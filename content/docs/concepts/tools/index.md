---
date: '2025-03-27T11:27:58+08:00'
draft: false
title: "工具"
description: "通过您的服务器使 LLMs 执行操作"
---

工具是模型上下文协议 (Model Context Protocol, MCP) 中一项强大的原语，允许服务器向客户端暴露可执行功能。借助工具，LLMs 可以与外部系统交互、执行计算，并在现实世界中采取行动。

> 工具被设计为**模型控制**，这意味着工具从服务器暴露给客户端，意图是让 AI 模型能够自动调用它们（包含人工审核以授予批准）。

## 概述

MCP 中的工具允许服务器暴露可被客户端调用的可执行功能，并可由 LLMs 用于执行操作。工具的关键特点包括：

- **发现**：客户端可以通过 `tools/list` 端点列出可用工具
- **调用**：工具通过 `tools/call` 端点被调用，服务器执行请求的操作并返回结果
- **灵活性**：工具可以从简单的计算到复杂的 API 交互

类似于[资源](/docs/concepts/resources)，工具通过唯一的名称进行标识，并可以包含说明以指导使用。然而，与资源不同的是，工具代表可以修改状态或与外部系统交互的动态操作。

## 工具定义结构

每个工具通过以下结构定义：

```typescript
{
  name: string;          // 工具的唯一标识符
  description?: string;  // 人类可读的描述
  inputSchema: {         // 工具参数的 JSON 架构
    type: "object",
    properties: { ... }  // 工具特定参数
  }
}
```

## 实现工具

以下是一个在 MCP 服务器中实现基础工具的示例：

{{< tabpane persist="lang" text=true >}}
{{% tab header="TypeScript" %}}

```typescript
const server = new Server({
  name: "example-server",
  version: "1.0.0"
}, {
  capabilities: {
    tools: {}
  }
});

// 定义可用工具
server.setRequestHandler(ListToolsRequestSchema, async () => {
  return {
    tools: [{
      name: "calculate_sum",
      description: "将两个数字相加",
      inputSchema: {
        type: "object",
        properties: {
          a: { type: "number" },
          b: { type: "number" }
        },
        required: ["a", "b"]
      }
    }]
  };
});

// 处理工具执行
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  if (request.params.name === "calculate_sum") {
    const { a, b } = request.params.arguments;
    return {
      content: [
        {
          type: "text",
          text: String(a + b)
        }
      ]
    };
  }
  throw new Error("工具未找到");
});

```

{{% /tab %}}
{{% tab header="Python" %}}

```python
app = Server("example-server")

@app.list_tools()
async def list_tools() -> list[types.Tool]:
    return [
        types.Tool(
            name="calculate_sum",
            description="将两个数字相加",
            inputSchema={
                "type": "object",
                "properties": {
                    "a": {"type": "number"},
                    "b": {"type": "number"}
                },
                "required": ["a", "b"]
            }
        )
    ]

@app.call_tool()
async def call_tool(
    name: str,
    arguments: dict
) -> list[types.TextContent | types.ImageContent | types.EmbeddedResource]:
    if name == "calculate_sum":
        a = arguments["a"]
        b = arguments["b"]
        result = a + b
        return [types.TextContent(type="text", text=str(result))]
    raise ValueError(f"未找到工具: {name}")
```

{{% /tab %}}
{{< /tabpane >}}

## 示例工具模式

以下是服务器可能提供的一些工具类型示例：

### 系统操作

与本地系统交互的工具：

```typescript
{
  name: "execute_command",
  description: "运行一个 shell 命令",
  inputSchema: {
    type: "object",
    properties: {
      command: { type: "string" },
      args: { type: "array", items: { type: "string" } }
    }
  }
}
```

### API 集成

封装外部 API 的工具：

```typescript
{
  name: "github_create_issue",
  description: "创建一个 GitHub issue",
  inputSchema: {
    type: "object",
    properties: {
      title: { type: "string" },
      body: { type: "string" },
      labels: { type: "array", items: { type: "string" } }
    }
  }
}
```

### 数据处理

用于转换或分析数据的工具：

```typescript
{
  name: "analyze_csv",
  description: "分析一个 CSV 文件",
  inputSchema: {
    type: "object",
    properties: {
      filepath: { type: "string" },
      operations: {
        type: "array",
        items: {
          enum: ["sum", "average", "count"]
        }
      }
    }
  }
}
```

## 最佳实践

实现工具时：

1. 提供清晰、描述性的名称和说明
2. 为参数使用详细的 JSON Schema 定义
3. 在工具说明中包含示例，展示模型如何使用它们
4. 实现适当的错误处理和验证
5. 对长时间运行的操作使用进度报告
6. 保持工具操作精简独立
7. 记录预期返回值结构
8. 实现适当的超时机制
9. 对资源密集型操作考虑速率限制
10. 记录工具使用情况以便调试和监控

## 安全考虑

暴露工具时：

### 输入验证

- 检查所有参数是否符合 schema
- 清理文件路径和系统命令
- 验证 URL 和外部标识符
- 检查参数大小和范围
- 防止命令注入

### 访问控制

- 实现必要的身份验证
- 使用适当的授权检查
- 审计工具使用情况
- 限制请求速率
- 监控滥用行为

### 错误处理

- 不向客户端暴露内部错误
- 记录与安全相关的错误
- 适当处理超时
- 在错误后清理资源
- 验证返回值

## 工具发现和更新

MCP 支持动态工具发现：

1. 客户端随时可以列出可用工具
2. 服务器可以通过 `notifications/tools/list_changed` 通知客户端工具变化
3. 工具可以在运行时添加或移除
4. 工具定义可以更新（但应谨慎进行）

## 错误处理

工具错误应在结果对象中报告，而不是作为 MCP 协议级别错误。这使得 LLM 可以查看并可能处理错误。当工具遇到错误时：

1. 在结果中将 `isError` 设置为 `true`
2. 在 `content` 数组中包含错误详细信息

以下是正确处理工具错误的示例：

{{< tabpane persist="lang" text=true >}}
{{% tab header="TypeScript" %}}

```typescript
try {
  // 工具操作
  const result = performOperation();
  return {
    content: [
      {
        type: "text",
        text: `操作成功: ${result}`
      }
    ]
  };
} catch (error) {
  return {
    isError: true,
    content: [
      {
        type: "text",
        text: `错误: ${error.message}`
      }
    ]
  };
}
```

{{% /tab %}}
{{% tab header="Python" %}}

```python
try:
    # 工具操作
    result = perform_operation()
    return types.CallToolResult(
        content=[
            types.TextContent(
                type="text",
                text=f"操作成功: {result}"
            )
        ]
    )
except Exception as error:
    return types.CallToolResult(
        isError=True,
        content=[
            types.TextContent(
                type="text",
                text=f"错误: {str(error)}"
            )
        ]
    )
```

{{% /tab %}}
{{< /tabpane >}}

此方法使得 LLM 可以看到发生了错误，并可能采取纠正措施或请求人工干预。

## 测试工具

MCP 工具的全面测试策略应包括：

- **功能测试**：验证工具在有效输入下正确执行，并妥善处理无效输入
- **集成测试**：使用真实或模拟的依赖项测试工具与外部系统的交互
- **安全测试**：验证身份验证、授权、输入清理和速率限制
- **性能测试**：检查负载下的行为、超时处理和资源清理
- **错误处理**：确保工具通过 MCP 协议正确报告错误并清理资源
