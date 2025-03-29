---
title: 基础协议
draft: false
weight: 20
---

> **Protocol Revision**: 2025-03-26

Model Context Protocol（MCP）由多个关键组件协同工作组成：

- **Base Protocol**（基础协议）：核心 JSON-RPC 消息类型
- **Lifecycle Management**（生命周期管理）：连接初始化、能力协商和会话控制
- **Server Features**（服务器功能）：服务器提供的资源、提示和工具
- **Client Features**（客户端功能）：客户端提供的采样和根目录列表
- **Utilities**（实用工具）：如日志记录和参数补全等跨模块需求

所有实现**必须**支持基础协议和生命周期管理组件。其他组件**可以**根据应用的具体需求来实现。

这些协议层提供了明确的关注点分离，同时在客户端和服务器之间实现了丰富的交互。模块化设计使得实现可以根据需求支持恰当的功能。

## 消息

MCP 客户端和服务器之间的所有消息**必须**遵循 [`JSON-RPC 2.0`](https://www.jsonrpc.org/specification) 规范。协议定义了以下消息类型：

### 请求（Requests）

客户端或服务器发送请求以启动操作。

```typescript
{
  jsonrpc: "2.0";
  id: string | number;
  method: string;
  params?: {
    [key: string]: unknown;
  };
}
```

- 请求**必须**包含字符串或整数 ID。
- 不同于基础 `JSON-RPC`，ID **不得** 为 `null`。
- 请求 ID **不得**在同一会话中被请求者重复使用。

### 响应（Responses）

响应用于回复请求，包含操作结果或错误信息。

```typescript
{
  jsonrpc: "2.0";
  id: string | number;
  result?: {
    [key: string]: unknown;
  }
  error?: {
    code: number;
    message: string;
    data?: unknown;
  }
}
```

- 响应**必须**包含与其对应请求相同的 ID。
- **响应**分为**成功结果**和**错误**，要么设置 `result`，要么设置 `error`，但**不得**同时设置两者。
- 结果**可以**遵循任意 JSON 对象结构，而错误**至少**包括错误代码和消息。
- 错误代码**必须**是整数。

### 通知（Notifications）

通知是从客户端或服务器发送的一种单向消息。接收者**不得**发送响应。

```typescript
{
  jsonrpc: "2.0";
  method: string;
  params?: {
    [key: string]: unknown;
  };
}
```

- 通知**不得**包含 ID。

### 批处理（Batching）

JSON-RPC 定义了一个方法来 [批量发送多个请求和通知](https://www.jsonrpc.org/specification#batch)，将它们以数组方式发送。MCP 实现**可以**支持发送 JSON-RPC 批处理，但**必须**支持接收 JSON-RPC 批处理。

## 认证（Auth）

MCP 提供了一个基于 HTTP 的 [Authorization](authorization) 框架。使用基于 HTTP 的传输的实现**应该**遵循此规范，而使用 STDIO 传输的实现 **不应该** 遵循此规范，而是从环境中检索凭据。

此外，客户端和服务器**可以**协商定制的认证和授权策略。

关于 MCP 认证机制演化的进一步讨论和贡献，请加入 [GitHub Discussions](https://github.com/modelcontextprotocol/specification/discussions)，共同助力协议未来的发展！

## 架构（Schema）

协议的完整规范定义为一个 [TypeScript 模式](http://github.com/modelcontextprotocol/specification/tree/main/schema/draft/schema.ts)。这是所有协议消息和结构的源头。

同时还生成了一个 [JSON Schema](http://github.com/modelcontextprotocol/specification/tree/main/schema/draft/schema.json)，它是从 TypeScript 源头自动生成的，可用于各种自动化工具。
~~~

