---
date: '2025-04-10T19:18:04+08:00'
draft: false
title: 'Python'
---

模型上下文协议（MCP）的 Python 实现

## 概览

模型上下文协议允许应用程序以标准化方式为大型语言模型（LLM）提供上下文，将提供上下文的关注点与实际的 LLM 交互分开。这个 Python SDK 实现了完整的 MCP 规范，使您可以轻松地：

- 构建能够连接到任何 MCP 服务器的 MCP 客户端
- 创建暴露资源、提示和工具的 MCP 服务器
- 使用标准传输协议，如 stdio 和 SSE
- 处理所有 MCP 协议消息和生命周期事件

## 安装

### 将 MCP 添加到您的 Python 项目

我们推荐使用 [uv](https://docs.astral.sh/uv/) 来管理您的 Python 项目。

如果您还没有创建 uv 管理的项目，请创建一个：

```bash
uv init mcp-server-demo
cd mcp-server-demo
```

然后将 MCP 添加到您的项目依赖中：

```bash
uv add "mcp[cli]"
```

或者，对于使用 pip 管理依赖的项目：

```bash
pip install "mcp[cli]"
```

### 运行独立的 MCP 开发工具

使用 uv 运行 mcp 命令：

```bash
uv run mcp
```

## 快速入门

让我们创建一个简单的 MCP 服务器，暴露一个计算器工具和一些数据：

```python
# server.py
from mcp.server.fastmcp import FastMCP

# 创建一个 MCP 服务器
mcp = FastMCP("Demo")

# 添加一个加法工具
@mcp.tool()
def add(a: int, b: int) -> int:
    """将两个数字相加"""
    return a + b

# 添加一个动态问候资源
@mcp.resource("greeting://{name}")
def get_greeting(name: str) -> str:
    """获取个性化问候"""
    return f"你好，{name}！"
```

您可以通过运行以下命令将此服务器安装到 [Claude Desktop](https://claude.ai/download) 并立即与之交互：

```bash
mcp install server.py
```

或者，您可以使用 MCP Inspector 测试它：

```bash
mcp dev server.py
```

## 什么是 MCP

[模型上下文协议（MCP）](https://modelcontextprotocol.io/) 让您可以构建以安全、标准化方式向 LLM 应用程序暴露数据和功能的服务器。可以将其想象为专为 LLM 交互设计的 Web API。MCP 服务器可以：

- 通过 资源 暴露数据（类似于 GET 端点；用于将信息加载到 LLM 的上下文中）
- 通过 工具 提供功能（类似于 POST 端点；用于执行代码或产生副作用）
- 通过 提示 定义交互模式（LLM 交互的可重用模板）
- 以及更多！

## 核心概念

### 服务器

FastMCP 服务器是您与 MCP 协议的核心接口。它处理连接管理、协议合规性和消息路由：

```python
# 添加具有强类型的启动/关闭生命周期支持
from contextlib import asynccontextmanager
from collections.abc import AsyncIterator
from dataclasses import dataclass

from fake_database import Database  # 替换为您的实际数据库类型

from mcp.server.fastmcp import Context, FastMCP

# 创建一个命名服务器
mcp = FastMCP("我的应用")

# 指定部署和开发的依赖
mcp = FastMCP("我的应用", dependencies=["pandas", "numpy"])


@dataclass
class AppContext:
    db: Database


@asynccontextmanager
async def app_lifespan(server: FastMCP) -> AsyncIterator[AppContext]:
    """使用类型安全的上下文管理应用生命周期"""
    # 启动时初始化
    db = await Database.connect()
    try:
        yield AppContext(db=db)
    finally:
        # 关闭时清理
        await db.disconnect()


# 将生命周期传递给服务器
mcp = FastMCP("我的应用", lifespan=app_lifespan)


# 在工具中访问类型安全的生命周期上下文
@mcp.tool()
def query_db(ctx: Context) -> str:
    """使用初始化资源的工具"""
    db = ctx.request_context.lifespan_context.db
    return db.query()
```

### 资源

资源是您向 LLM 暴露数据的方式。它们类似于 REST API 中的 GET 端点——提供数据，但不应执行重大计算或产生副作用：

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("我的应用")


@mcp.resource("config://app")
def get_config() -> str:
    """静态配置数据"""
    return "应用配置在此"


@mcp.resource("users://{user_id}/profile")
def get_user_profile(user_id: str) -> str:
    """动态用户数据"""
    return f"用户 {user_id} 的个人资料数据"
```

### 工具

工具允许 LLM 通过您的服务器执行操作。与资源不同，工具预期会执行计算并产生副作用：

```python
import httpx
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("我的应用")


@mcp.tool()
def calculate_bmi(weight_kg: float, height_m: float) -> float:
    """根据体重（公斤）和身高（米）计算 BMI"""
    return weight_kg / (height_m**2)


@mcp.tool()
async def fetch_weather(city: str) -> str:
    """获取城市的当前天气"""
    async with httpx.AsyncClient() as client:
        response = await client.get(f"https://api.weather.com/{city}")
        return response.text
```

### 提示

提示是可重用的模板，帮助 LLM 有效地与您的服务器交互：

```python
from mcp.server.fastmcp import FastMCP
from mcp.server.fastmcp.prompts import base

mcp = FastMCP("我的应用")


@mcp.prompt()
def review_code(code: str) -> str:
    return f"请审查这段代码：\n\n{code}"


@mcp.prompt()
def debug_error(error: str) -> list[base.Message]:
    return [
        base.UserMessage("我遇到了这个错误："),
        base.UserMessage(error),
        base.AssistantMessage("我将帮助调试。你到目前为止尝试了什么？"),
    ]
```

### 图像

FastMCP 提供了一个 `Image` 类，自动处理图像数据：

```python
from mcp.server.fastmcp import FastMCP, Image
from PIL import Image as PILImage

mcp = FastMCP("我的应用")


@mcp.tool()
def create_thumbnail(image_path: str) -> Image:
    """从图像创建缩略图"""
    img = PILImage.open(image_path)
    img.thumbnail((100, 100))
    return Image(data=img.tobytes(), format="png")
```

### 上下文

Context 对象为您的工具和资源提供了对 MCP 功能的访问：

```python
from mcp.server.fastmcp import FastMCP, Context

mcp = FastMCP("我的应用")


@mcp.tool()
async def long_task(files: list[str], ctx: Context) -> str:
    """处理多个文件并跟踪进度"""
    for i, file in enumerate(files):
        ctx.info(f"正在处理 {file}")
        await ctx.report_progress(i, len(files))
        data, mime_type = await ctx.read_resource(f"file://{file}")
    return "处理完成"
```

## 运行服务器

### 开发模式

测试和调试服务器的最快方法是使用 MCP Inspector：

```bash
mcp dev server.py

# 添加依赖
mcp dev server.py --with pandas --with numpy

# 挂载本地代码
mcp dev server.py --with-editable .
```

### Claude 桌面

一旦您的服务器准备就绪，可以将其安装到 Claude Desktop 中：

```bash
mcp install server.py

# 自定义名称
mcp install server.py --name "我的分析服务器"

# 环境变量
mcp install server.py -v API_KEY=abc123 -v DB_URL=postgres://...
mcp install server.py -f .env
```

### 直接执行

对于自定义部署等高级场景：

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("我的应用")

if __name__ == "__main__":
    mcp.run()
```

运行它：

```bash
python server.py
# 或
mcp run server.py
```

### 挂载到现有的 ASGI 服务器

您可以使用 `sse_app` 方法将 SSE 服务器挂载到现有的 ASGI 服务器上。这允许您将 SSE 服务器与其他 ASGI 应用程序集成。

```python
from starlette.applications import Starlette
from starlette.routing import Mount, Host
from mcp.server.fastmcp import FastMCP


mcp = FastMCP("我的应用")

# 将 SSE 服务器挂载到现有的 ASGI 服务器
app = Starlette(
    routes=[
        Mount('/', app=mcp.sse_app()),
    ]
)

# 或动态挂载为主机
app.router.routes.append(Host('mcp.acme.corp', app=mcp.sse_app()))
```

有关在 Starlette 中挂载应用程序的更多信息，请参阅 [Starlette 文档](https://www.starlette.io/routing/#submounting-routes)。

## 示例

### Echo 服务

一个简单的服务器，展示资源、工具和提示：

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("Echo")

@mcp.resource("echo://{message}")
def echo_resource(message: str) -> str:
    """Echo a message as a resource"""
    return f"Resource echo: {message}"

@mcp.tool()
def echo_tool(message: str) -> str:
    """Echo a message as a tool"""
    return f"Tool echo: {message}"

@mcp.prompt()
def echo_prompt(message: str) -> str:
    """Create an echo prompt"""
    return f"Please process this message: {message}"
```

### SQLite 浏览器

一个更复杂的示例，展示集成数据库的能力：

```python
import sqlite3

from mcp.server.fastmcp import FastMCP

mcp = FastMCP("SQLite 浏览器")


@mcp.resource("schema://main")
def get_schema() -> str:
    """作为资源提供数据库模式"""
    conn = sqlite3.connect("database.db")
    schema = conn.execute("SELECT sql FROM sqlite_master WHERE type='table'").fetchall()
    return "\n".join(sql[0] for sql in schema if sql[0])


@mcp.tool()
def query_data(sql: str) -> str:
    """安全执行 SQL 查询"""
    conn = sqlite3.connect("database.db")
    try:
        result = conn.execute(sql).fetchall()
        return "\n".join(str(row) for row in result)
    except Exception as e:
        return f"错误：{str(e)}"
```

## 高级用法

### 底层服务

为了获得更多控制，您可以直接使用底层服务实现。这为您提供了对协议的完全访问，并允许您自定义服务器的每个方面，包括通过 Lifecycle API 管理生命周期：

```python
from contextlib import asynccontextmanager
from collections.abc import AsyncIterator

from fake_database import Database  # 替换为您的实际数据库类型

from mcp.server import Server


@asynccontextmanager
async def server_lifespan(server: Server) -> AsyncIterator[dict]:
    """管理服务器启动和关闭生命周期"""
    # 启动时初始化资源
    db = await Database.connect()
    try:
        yield {"db": db}
    finally:
        # 关闭时清理
        await db.disconnect()


# 将生命周期传递给服务器
server = Server("example-server", lifespan=server_lifespan)


# 在处理程序中访问生命周期上下文
@server.call_tool()
async def query_db(name: str, arguments: dict) -> list:
    ctx = server.request_context
    db = ctx.lifespan_context["db"]
    return await db.query(arguments["query"])
```

Lifecycle API 提供：

- 在服务器启动时初始化资源并在停止时清理它们的方法
- 通过请求上下文在处理程序中访问初始化资源
- 在生命周期和请求处理程序之间传递类型安全的上下文

```python
import mcp.server.stdio
import mcp.types as types
from mcp.server.lowlevel import NotificationOptions, Server
from mcp.server.models import InitializationOptions

# 创建服务器实例
server = Server("example-server")


@server.list_prompts()
async def handle_list_prompts() -> list[types.Prompt]:
    return [
        types.Prompt(
            name="example-prompt",
            description="示例提示模板",
            arguments=[
                types.PromptArgument(
                    name="arg1", description="示例参数", required=True
                )
            ],
        )
    ]


@server.get_prompt()
async def handle_get_prompt(
    name: str, arguments: dict[str, str] | None
) -> types.GetPromptResult:
    if name != "example-prompt":
        raise ValueError(f"未知提示：{name}")

    return types.GetPromptResult(
        description="示例提示",
        messages=[
            types.PromptMessage(
                role="user",
                content=types.TextContent(type="text", text="示例提示文本"),
            )
        ],
    )


async def run():
    async with mcp.server.stdio.stdio_server() as (read_stream, write_stream):
        await server.run(
            read_stream,
            write_stream,
            InitializationOptions(
                server_name="example",
                server_version="0.1.0",
                capabilities=server.get_capabilities(
                    notification_options=NotificationOptions(),
                    experimental_capabilities={},
                ),
            ),
        )


if __name__ == "__main__":
    import asyncio

    asyncio.run(run())
```

## 编写 MCP 客户端

SDK 提供了一个高级客户端接口，用于连接到 MCP 服务器：

```python
from mcp import ClientSession, StdioServerParameters, types
from mcp.client.stdio import stdio_client

# 为 stdio 连接创建服务器参数
server_params = StdioServerParameters(
    command="python",  # 可执行文件
    args=["example_server.py"],  # 可选命令行参数
    env=None,  # 可选环境变量
)


# 可选：创建采样回调
async def handle_sampling_message(
    message: types.CreateMessageRequestParams,
) -> types.CreateMessageResult:
    return types.CreateMessageResult(
        role="assistant",
        content=types.TextContent(
            type="text",
            text="你好，世界！来自模型",
        ),
        model="gpt-3.5-turbo",
        stopReason="endTurn",
    )


async def run():
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(
            read, write, sampling_callback=handle_sampling_message
        ) as session:
            # 初始化连接
            await session.initialize()

            # 列出可用提示
            prompts = await session.list_prompts()

            # 获取提示
            prompt = await session.get_prompt(
                "example-prompt", arguments={"arg1": "value"}
            )

            # 列出可用资源
            resources = await session.list_resources()

            # 列出可用工具
            tools = await session.list_tools()

            # 读取资源
            content, mime_type = await session.read_resource("file://some/path")

            # 调用工具
            result = await session.call_tool("tool-name", arguments={"arg1": "value"})


if __name__ == "__main__":
    import asyncio

    asyncio.run(run())
```

## MCP 原语

MCP 协议定义了服务器可以实现的三种核心原语：

|原语|控制权|描述|示例用途|
|---|---|---|---|
|提示|用户控制|用户选择调用的交互式模板|斜杠命令，菜单选项|
|资源|应用程序控制|客户端应用程序管理的上下文数据|文件内容，API 响应|
|工具|模型控制|暴露给 LLM 以执行操作的功能|API 调用，数据更新|

## 服务端 Capability

MCP 服务器在初始化时声明 Capability：

|功能|特性标志|描述|
|---|---|---|
|prompts|listChanged|提示模板管理|
|resources|subscribe<br/>listChanged|资源暴露和更新|
|tools|listChanged|工具发现和执行|
|logging|-|服务器日志配置|
|completion|-|参数补全建议|

## 文档

- [模型上下文协议文档](https://modelcontextprotocol.io/)
- [模型上下文协议规范](https://spec.modelcontextprotocol.io/)
- [官方支持的服务器](https://github.com/modelcontextprotocol/servers)

## 贡献

我们热衷于支持各种经验水平的贡献者，并希望看到您参与到项目中。请参阅  开始。

## 许可

该项目采用 MIT 许可证授权 - 详情请参阅 LICENSE 文件。
