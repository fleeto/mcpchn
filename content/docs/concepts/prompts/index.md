---
date: '2025-03-27T11:20:51+08:00'
draft: false
title: "Prompts"
description: "创建可重用的提示模板和工作流程"
---

提示（Prompts）使服务器能够定义可重用的提示模板和工作流程，客户端可以将其轻松呈现给用户和 LLMs（大语言模型）。它们提供了一种强大的方式来标准化和共享常见的 LLM 交互方式。

> 提示的设计是 **用户控制** 的，这意味着它们从服务器暴露给客户端时，目的是让用户能够显式选择使用这些提示。

## 概述

MCP（Model Context Protocol）的提示是预定义的模板，可以：

- 接受动态参数
- 包含资源的上下文
- 链接多个交互步骤
- 指导特定的工作流程
- 作为 UI 元素展示（例如斜杠命令）

## 提示结构

每个提示通过以下方式定义：

```typescript
{
  name: string;              // 提示的唯一标识符
  description?: string;      // 人类可读的描述
  arguments?: [              // 可选的参数列表
    {
      name: string;          // 参数标识符
      description?: string;  // 参数描述
      required?: boolean;    // 参数是否为必需
    }
  ]
}
```

## 发现提示

客户端可以通过 `prompts/list` 接口发现可用的提示：

```typescript
// 请求
{
  method: "prompts/list"
}

// 响应
{
  prompts: [
    {
      name: "analyze-code",
      description: "分析代码以寻找潜在的改进",
      arguments: [
        {
          name: "language",
          description: "编程语言",
          required: true
        }
      ]
    }
  ]
}
```

## 使用提示

要使用提示，客户端可以发起 `prompts/get` 请求：

```typescript
// 请求
{
  method: "prompts/get",
  params: {
    name: "analyze-code",
    arguments: {
      language: "python"
    }
  }
}

// 响应
{
  description: "分析 Python 代码以寻找潜在的改进",
  messages: [
    {
      role: "user",
      content: {
        type: "text",
        text: "请分析以下 Python 代码并寻找潜在的改进：\n\n```python\ndef calculate_sum(numbers):\n    total = 0\n    for num in numbers:\n        total = total + num\n    return total\n\nresult = calculate_sum([1, 2, 3, 4, 5])\nprint(result)\n```"
      }
    }
  ]
}
```

## 动态提示

提示可以是动态的，并包括以下内容：

### 嵌入的资源上下文

```json
{
  "name": "analyze-project",
  "description": "分析项目日志和代码",
  "arguments": [
    {
      "name": "timeframe",
      "description": "用于分析日志的时间范围",
      "required": true
    },
    {
      "name": "fileUri",
      "description": "需要审查的代码文件的 URI",
      "required": true
    }
  ]
}
```

在处理 `prompts/get` 请求时：

```json
{
  "messages": [
    {
      "role": "user",
      "content": {
        "type": "text",
        "text": "请分析这些系统日志和代码文件是否存在问题："
      }
    },
    {
      "role": "user",
      "content": {
        "type": "resource",
        "resource": {
          "uri": "logs://recent?timeframe=1h",
          "text": "[2024-03-14 15:32:11] ERROR: Connection timeout in network.py:127\n[2024-03-14 15:32:15] WARN: Retrying connection (attempt 2/3)\n[2024-03-14 15:32:20] ERROR: Max retries exceeded",
          "mimeType": "text/plain"
        }
      }
    },
    {
      "role": "user",
      "content": {
        "type": "resource",
        "resource": {
          "uri": "file:///path/to/code.py",
          "text": "def connect_to_service(timeout=30):\n    retries = 3\n    for attempt in range(retries):\n        try:\n            return establish_connection(timeout)\n        except TimeoutError:\n            if attempt == retries - 1:\n                raise\n            time.sleep(5)\n\ndef establish_connection(timeout):\n    # Connection implementation\n    pass",
          "mimeType": "text/x-python"
        }
      }
    }
  ]
}
```

### 多步骤工作流程

```typescript
const debugWorkflow = {
  name: "debug-error",
  async getMessages(error: string) {
    return [
      {
        role: "user",
        content: {
          type: "text",
          text: `这是我遇到的一个错误：${error}`
        }
      },
      {
        role: "assistant",
        content: {
          type: "text",
          text: "我将帮助分析该错误。请问您已经尝试过什么方法？"
        }
      },
      {
        role: "user",
        content: {
          type: "text",
          text: "我尝试过重新启动该服务，但问题仍然存在。"
        }
      }
    ];
  }
};
```

## 示例实现

以下是实现 MCP 服务器中提示功能的完整示例：

{{< tabpane persist="lang" text=true >}}
{{% tab header="TypeScript" %}}

```typescript
import { Server } from "@modelcontextprotocol/sdk/server";
import {
  ListPromptsRequestSchema,
  GetPromptRequestSchema
} from "@modelcontextprotocol/sdk/types";

const PROMPTS = {
  "git-commit": {
    name: "git-commit",
    description: "生成 Git 提交信息",
    arguments: [
      {
        name: "changes",
        description: "Git diff 或更改的描述",
        required: true
      }
    ]
  },
  "explain-code": {
    name: "explain-code",
    description: "解释代码的工作原理",
    arguments: [
      {
        name: "code",
        description: "需要解释的代码",
        required: true
      },
      {
        name: "language",
        description: "编程语言",
        required: false
      }
    ]
  }
};

const server = new Server({
  name: "example-prompts-server",
  version: "1.0.0"
}, {
  capabilities: {
    prompts: {}
  }
});

// 列出可用的提示
server.setRequestHandler(ListPromptsRequestSchema, async () => {
  return {
    prompts: Object.values(PROMPTS)
  };
});

// 获取指定的提示
server.setRequestHandler(GetPromptRequestSchema, async (request) => {
  const prompt = PROMPTS[request.params.name];
  if (!prompt) {
    throw new Error(`找不到提示：${request.params.name}`);
  }

  if (request.params.name === "git-commit") {
    return {
      messages: [
        {
          role: "user",
          content: {
            type: "text",
            text: `根据以下更改生成一条简洁而具有描述性的提交信息：\n\n${request.params.arguments?.changes}`
          }
        }
      ]
    };
  }

  if (request.params.name === "explain-code") {
    const language = request.params.arguments?.language || "未知";
    return {
      messages: [
        {
          role: "user",
          content: {
            type: "text",
            text: `解释这段 ${language} 代码的工作原理：\n\n${request.params.arguments?.code}`
          }
        }
      ]
    };
  }

  throw new Error("未找到提示实现");
});
```

{{% /tab %}}
{{% tab header="Python" %}}

```python
from mcp.server import Server
import mcp.types as types

# 定义可用的提示
PROMPTS = {
    "git-commit": types.Prompt(
        name="git-commit",
        description="生成 Git 提交信息",
        arguments=[
            types.PromptArgument(
                name="changes",
                description="Git diff 或更改的描述",
                required=True
            )
        ],
    ),
    "explain-code": types.Prompt(
        name="explain-code",
        description="解释代码的工作原理",
        arguments=[
            types.PromptArgument(
                name="code",
                description="需要解释的代码",
                required=True
            ),
            types.PromptArgument(
                name="language",
                description="编程语言",
                required=False
            )
        ],
    )
}

# 初始化服务器
app = Server("example-prompts-server")

@app.list_prompts()
async def list_prompts() -> list[types.Prompt]:
    return list(PROMPTS.values())

@app.get_prompt()
async def get_prompt(
    name: str, arguments: dict[str, str] | None = None
) -> types.GetPromptResult:
    if name not in PROMPTS:
        raise ValueError(f"找不到提示：{name}")

    if name == "git-commit":
        changes = arguments.get("changes") if arguments else ""
        return types.GetPromptResult(
            messages=[
                types.PromptMessage(
                    role="user",
                    content=types.TextContent(
                        type="text",
                        text=f"根据以下更改生成一条简洁而具有描述性的提交信息：\n\n{changes}"
                    )
                )
            ]
        )

    if name == "explain-code":
        code = arguments.get("code") if arguments else ""
        language = arguments.get("language", "未知") if arguments else "未知"
        return types.GetPromptResult(
            messages=[
                types.PromptMessage(
                    role="user",
                    content=types.TextContent(
                        type="text",
                        text=f"解释这段 {language} 代码的工作原理：\n\n{code}"
                    )
                )
            ]
        )

    raise ValueError("未找到提示实现")
```

{{% /tab %}}
{{< /tabpane >}}

## 最佳实践

在实现提示功能时：

1. 使用清晰且具有描述性的提示名称
2. 为提示及参数提供详细描述
3. 验证所有必填参数
4. 优雅地处理缺少的参数
5. 考虑提示模板的版本化
6. 在适当情况下缓存动态内容
7. 实现错误处理机制
8. 文档化参数的预期格式
9. 考虑提示的可组合性
10. 用各种输入测试提示功能

## UI 集成

提示可以以下列方式在客户端 UI 中呈现：

- 斜杠命令
- 快捷操作
- 上下文菜单选项
- 命令面板条目
- 引导式工作流程
- 交互式表单

## 更新与变更

服务器可以通过以下方式通知客户端提示的变更：

1. 服务器能力：`prompts.listChanged`
2. 通知：`notifications/prompts/list_changed`
3. 客户端重新获取提示列表

## 安全注意事项

在实现提示功能时需关注：

- 验证所有参数
- 清理用户输入
- 考虑速率限制
- 实现访问控制
- 审计提示的使用情况
- 适当地处理敏感数据
- 验证生成内容
- 实现超时机制
- 考虑提示注入风险
- 文档化安全需求
