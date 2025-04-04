---
date: '2025-03-27T11:47:22+08:00'
draft: false
title: "传输机制"
description: "了解 MCP 的通信机制"
---

Model Context Protocol (MCP) 中的传输机制为客户端与服务器之间的通信提供了基础。传输层负责管理消息的发送和接收的底层机制。

## 消息格式

MCP 使用 [JSON-RPC](https://www.jsonrpc.org/) 2.0 作为其数据传输格式。传输层负责将 MCP 协议消息转换为 JSON-RPC 格式进行传输，并将接收到的 JSON-RPC 消息转换回 MCP 协议消息。

JSON-RPC 消息分为三种类型：

### 请求 (Requests)

```typescript
{
  jsonrpc: "2.0",
  id: number | string,
  method: string,
  params?: object
}
```

### 响应 (Responses)

```typescript
{
  jsonrpc: "2.0",
  id: number | string,
  result?: object,
  error?: {
    code: number,
    message: string,
    data?: unknown
  }
}
```

### 通知 (Notifications)

```typescript
{
  jsonrpc: "2.0",
  method: string,
  params?: object
}
```

## 内置传输类型

MCP 包含两种标准的传输实现：

### 标准输入/输出 (stdio)

`stdio` 传输通过标准输入和输出流进行通信。这对本地集成和命令行工具特别有用。

使用 `stdio` 的场景：

- 创建命令行工具
- 实现本地集成
- 需要简单的进程通信
- 在 Shell 脚本中工作

{{< tabpane persist="lang" text=true >}}
{{% tab header="TypeScript（服务端）" %}}

```typescript
const server = new Server({
  name: "example-server",
  version: "1.0.0"
}, {
  capabilities: {}
});

const transport = new StdioServerTransport();
await server.connect(transport);
```

{{% /tab %}}
{{% tab header="TypeScript（客户端）" %}}

```typescript
const client = new Client({
  name: "example-client",
  version: "1.0.0"
}, {
  capabilities: {}
});

const transport = new StdioClientTransport({
  command: "./server",
  args: ["--option", "value"]
});
await client.connect(transport);
```

{{% /tab %}}
{{% tab header="Python（服务端）" %}}

```python
app = Server("example-server")

async with stdio_server() as streams:
    await app.run(
        streams[0],
        streams[1],
        app.create_initialization_options()
    )
```

{{% /tab %}}
{{% tab header="Python（客户端）" %}}

```python
params = StdioServerParameters(
    command="./server",
    args=["--option", "value"]
)

async with stdio_client(params) as streams:
    async with ClientSession(streams[0], streams[1]) as session:
        await session.initialize()
```

{{% /tab %}}
{{< /tabpane >}}

### 服务器推送事件 (Server-Sent Events, SSE)

`SSE` 传输通过 HTTP POST 请求实现客户端到服务器的通信，并支持服务器到客户端的流式推送。

使用 SSE 的场景：

- 仅需要服务器到客户端的流式推送
- 适应受限网络环境
- 实现简单的更新机制

{{< tabpane persist="lang" text=true >}}
{{% tab header="TypeScript（服务端）" %}}

```typescript
import express from "express";

const app = express();

const server = new Server({
  name: "example-server",
  version: "1.0.0"
}, {
  capabilities: {}
});

let transport: SSEServerTransport | null = null;

app.get("/sse", (req, res) => {
  transport = new SSEServerTransport("/messages", res);
  server.connect(transport);
});

app.post("/messages", (req, res) => {
  if (transport) {
    transport.handlePostMessage(req, res);
  }
});

app.listen(3000);
```

{{% /tab %}}
{{% tab header="TypeScript（客户端）" %}}

```typescript
const client = new Client({
  name: "example-client",
  version: "1.0.0"
}, {
  capabilities: {}
});

const transport = new SSEClientTransport(
  new URL("http://localhost:3000/sse")
);
await client.connect(transport);
```

{{% /tab %}}
{{% tab header="Python（服务端）" %}}

```python
from mcp.server.sse import SseServerTransport
from starlette.applications import Starlette
from starlette.routing import Route

app = Server("example-server")
sse = SseServerTransport("/messages")

async def handle_sse(scope, receive, send):
    async with sse.connect_sse(scope, receive, send) as streams:
        await app.run(streams[0], streams[1], app.create_initialization_options())

async def handle_messages(scope, receive, send):
    await sse.handle_post_message(scope, receive, send)

starlette_app = Starlette(
    routes=[
        Route("/sse", endpoint=handle_sse),
        Route("/messages", endpoint=handle_messages, methods=["POST"]),
    ]
)
```

{{% /tab %}}
{{% tab header="Python（客户端）" %}}

```python
async with sse_client("http://localhost:8000/sse") as streams:
    async with ClientSession(streams[0], streams[1]) as session:
        await session.initialize()
```

{{% /tab %}}
{{< /tabpane >}}

## 自定义传输

MCP 允许根据特定需求实现自定义传输。任何传输实现都需要符合 `Transport` 接口：

您可以为以下场景设计自定义传输：

- 自定义网络协议
- 特殊通信渠道
- 与现有系统的集成
- 性能优化

{{< tabpane persist="lang" text=true >}}
{{% tab header="TypeScript" %}}

```typescript
interface Transport {
  // 开始处理消息
  start(): Promise<void>;

  // 发送 JSON-RPC 消息
  send(message: JSONRPCMessage): Promise<void>;

  // 关闭连接
  close(): Promise<void>;

  // 回调函数
  onclose?: () => void;
  onerror?: (error: Error) => void;
  onmessage?: (message: JSONRPCMessage) => void;
}
```

{{% /tab %}}
{{% tab header="Python" %}}

注意，虽然 MCP 服务器通常使用 asyncio 实现，但我们推荐使用 `anyio` 实现低级接口（如传输），以获得更广泛的兼容性。

```python
@contextmanager
async def create_transport(
    read_stream: MemoryObjectReceiveStream[JSONRPCMessage | Exception],
    write_stream: MemoryObjectSendStream[JSONRPCMessage]
):
    """
    MCP 的传输接口。

    Args:
        read_stream: 读取传入消息的流
        write_stream: 写出发送消息的流
    """
    async with anyio.create_task_group() as tg:
        try:
            # 开始处理消息
            tg.start_soon(lambda: process_messages(read_stream))

            # 发送消息
            async with write_stream:
                yield write_stream

        except Exception as exc:
            # 处理错误
            raise exc
        finally:
            # 清理工作
            tg.cancel_scope.cancel()
            await write_stream.aclose()
            await read_stream.aclose()
```

{{% /tab %}}
{{< /tabpane >}}

## 错误处理

传输实现需要处理多种错误场景：

1. 连接错误
2. 消息解析错误
3. 协议错误
4. 网络超时
5. 资源清理

错误处理示例：

{{< tabpane persist="lang" text=true >}}
{{% tab header="TypeScript" %}}

```typescript
class ExampleTransport implements Transport {
  async start() {
    try {
      // 连接逻辑
    } catch (error) {
      this.onerror?.(new Error(`连接失败: ${error}`));
      throw error;
    }
  }

  async send(message: JSONRPCMessage) {
    try {
      // 发送逻辑
    } catch (error) {
      this.onerror?.(new Error(`消息发送失败: ${error}`));
      throw error;
    }
  }
}
```

{{% /tab %}}
{{% tab header="Python" %}}

注意，虽然 MCP 服务器通常使用 `asyncio` 实现，但我们推荐使用 `anyio` 实现低级接口（如传输），以获得更广泛的兼容性。

```python
@contextmanager
async def example_transport(scope: Scope, receive: Receive, send: Send):
    try:
        # 创建双向通信的流
        read_stream_writer, read_stream = anyio.create_memory_object_stream(0)
        write_stream, write_stream_reader = anyio.create_memory_object_stream(0)

        async def message_handler():
            try:
                async with read_stream_writer:
                    # 消息处理逻辑
                    pass
            except Exception as exc:
                logger.error(f"消息处理失败: {exc}")
                raise exc

        async with anyio.create_task_group() as tg:
            tg.start_soon(message_handler)
            try:
                # 提供通信流
                yield read_stream, write_stream
            except Exception as exc:
                logger.error(f"传输错误: {exc}")
                raise exc
            finally:
                tg.cancel_scope.cancel()
                await write_stream.aclose()
                await read_stream.aclose()
    except Exception as exc:
        logger.error(f"初始化传输失败: {exc}")
        raise exc
```

{{% /tab %}}
{{< /tabpane >}}

## 最佳实践

在实现或使用 MCP 传输时：

1. 正确管理连接生命周期
2. 实现完善的错误处理
3. 在连接关闭时清理资源
4. 使用合适的超时设定
5. 在发送前验证消息
6. 记录传输事件以便调试
7. 实现必要的重新连接逻辑
8. 处理消息队列中的反压
9. 监控连接健康状态
10. 实施适当的安全措施

## 安全注意事项

在实现传输时：

### 认证与授权

- 实现正确的认证机制
- 验证客户端凭据
- 采用安全的令牌处理
- 实施授权验证

### 数据安全

- 使用 TLS 进行网络传输
- 加密敏感数据
- 验证消息完整性
- 设置消息大小限制
- 对输入数据进行清理

### 网络安全

- 实施速率限制
- 使用适当的超时设置
- 处理拒绝服务场景
- 监控异常模式
- 实施正确的防火墙规则

## 调试传输

传输问题调试技巧：

1. 启用调试日志
2. 监控消息流
3. 检查连接状态
4. 验证消息格式
5. 测试错误场景
6. 使用网络分析工具
7. 实现健康检查
8. 监控资源使用情况
9. 测试边界情况
10. 使用正确的错误跟踪机制
