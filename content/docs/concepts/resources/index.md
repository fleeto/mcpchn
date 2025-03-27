---
date: '2025-03-27T10:12:08+08:00'
draft: false
title: "资源"
description: "将服务器中的数据和内容暴露给LLMs"
---

资源是模型上下文协议（Model Context Protocol, MCP）的核心原语，它允许服务器暴露数据和内容供客户端读取，并作为 LLM 交互的上下文。

资源是被**应用控制**的，这意味着客户端应用程序可以决定如何以及何时使用它们。

不同的 MCP 客户端可能会以不同的方式处理资源。例如：

 - Claude Desktop当前要求用户显式选择资源后才能使用
 - 其他客户端可能会基于启发式方法自动选择资源
 - 某些实现可能甚至允许AI模型自己决定使用哪些资源

 服务器开发者在实现资源支持时，应准备好处理这些交互模式中的任何一种。若希望自动向模型暴露数据，服务器开发者应该使用**模型控制**的原语，例如 [Tools](./tools)。

## 概述

资源代表 MCP 服务器希望供客户端使用的任何类型的数据。这些数据可以包括：

- 文件内容
- 数据库记录
- API响应
- 实时系统数据
- 截图与图片
- 日志文件
- 以及更多

每个资源都通过一个唯一的URI标识，并且可以包含文本或二进制数据。

## 资源URI

资源通过以下格式的URI进行标识：`[protocol]://[host]/[path]`

例如：

- `file:///home/user/documents/report.pdf`
- `postgres://database/customers/schema`
- `screen://localhost/display1`

协议和路径结构由 MCP 服务器实现定义。服务器可以定义自己的自定义URI方案。

## 资源类型

资源可以包含两种内容类型：

### 文本资源

文本资源包含UTF-8编码的文本数据，适用于：

- 源代码
- 配置文件
- 日志文件
- JSON/XML 数据
- 纯文本

### 二进制资源

二进制资源包含以base64编码的原始二进制数据，适用于：

- 图片
- PDF文件
- 音频文件
- 视频文件
- 其他非文本格式

## 资源发现

客户端可以通过两种主要方法发现可用资源：

### 直接资源

服务器通过 `resources/list` 端点暴露具体资源列表。每个资源包括：

```typescript
{
  uri: string;           // 资源的唯一标识符
  name: string;          // 可读名称
  description?: string;  // 可选描述
  mimeType?: string;     // 可选的MIME类型
}
```

### 资源模板

对于动态资源，服务器可以暴露 [URI 模板](https://datatracker.ietf.org/doc/html/rfc6570)，客户端可以使用这些模板生成有效的资源URI：

```typescript
{
  uriTemplate: string;   // 符合RFC 6570的URI模板
  name: string;          // 此类型的可读名称
  description?: string;  // 可选描述
  mimeType?: string;     // 所有匹配资源的可选MIME类型
}
```

## 读取资源

要读取资源，客户端发送带有资源 URI 的 `resources/read` 请求。

服务器返回一个资源内容的列表：

```typescript
{
  contents: [
    {
      uri: string;        // 资源的URI
      mimeType?: string;  // 可选的MIME类型

      // 以下之一：
      text?: string;      // 文本资源内容
      blob?: string;      // 二进制资源内容（base64编码）
    }
  ]
}
```

> 服务器可以在响应单个`resources/read`请求时返回多个资源。例如，读取一个目录时可以返回其中的文件列表。

## 资源更新

MCP 通过两种机制支持资源的实时更新：

### 列表更改

服务器可以通过 `notifications/resources/list_changed` 通知客户端其可用资源列表已更改。

### 内容更改

客户端可以订阅特定资源的更新：

1. 客户端发送带有资源 URI 的 `resources/subscribe`
2. 服务器在资源更改时发送 `notifications/resources/updated`
3. 客户端通过 `resources/read` 获取最新内容
4. 客户端可以通过 `resources/unsubscribe` 取消订阅

## 实现示例

以下是 MCP 服务器中实现资源支持的简单示例：

{{< tabpane persist="lang" text=true >}}

{{% tab header="TypeScript" %}}

```typescript
const server = new Server({
  name: "example-server",
  version: "1.0.0"
}, {
  capabilities: {
    resources: {}
  }
});

// 列出可用资源
server.setRequestHandler(ListResourcesRequestSchema, async () => {
  return {
    resources: [
      {
        uri: "file:///logs/app.log",
        name: "Application Logs",
        mimeType: "text/plain"
      }
    ]
  };
});

// 读取资源内容
server.setRequestHandler(ReadResourceRequestSchema, async (request) => {
  const uri = request.params.uri;

  if (uri === "file:///logs/app.log") {
    const logContents = await readLogFile();
    return {
      contents: [
        {
          uri,
          mimeType: "text/plain",
          text: logContents
        }
      ]
    };
  }

  throw new Error("Resource not found");
});
```

{{% /tab %}}
{{% tab header="Python" %}}

```python
app = Server("example-server")

@app.list_resources()
async def list_resources() -> list[types.Resource]:
    return [
        types.Resource(
            uri="file:///logs/app.log",
            name="Application Logs",
            mimeType="text/plain"
        )
    ]

@app.read_resource()
async def read_resource(uri: AnyUrl) -> str:
    if str(uri) == "file:///logs/app.log":
        log_contents = await read_log_file()
        return log_contents

    raise ValueError("Resource not found")

# 启动服务器
async with stdio_server() as streams:
    await app.run(
        streams[0],
        streams[1],
        app.create_initialization_options()
    )
```

{{% /tab %}}
{{< /tabpane >}}

## 最佳实践

在实现资源支持时：

1. 使用清晰、描述性的资源名称和 URI
2. 提供有帮助的描述以指导 LLM 的理解
3. 设置已知的适当 MIME 类型
4. 对于动态内容实施资源模板
5. 对频繁变化的资源使用订阅
6. 使用清晰的错误消息优雅地处理错误
7. 对于大型资源列表考虑分页
8. 在适当时缓存资源内容
9. 在处理前验证 URI
10. 记录自定义的 URI 方案

## 安全注意事项

在暴露资源时：

- 验证所有资源 URI
- 实施适当的访问控制
- 清理文件路径以防目录遍历
- 谨慎处理二进制数据
- 对资源读取实施速率限制
- 审计资源访问
- 在传输中加密敏感数据
- 验证 MIME 类型
- 对长时间读取操作实现超时机制
- 适当地处理资源清理
