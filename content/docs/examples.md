---
date: '2025-03-26T21:53:18+08:00'
draft: false
title: 示例服务器
weight: 20
description: '示例服务器和实现列表'
---

本页面展示了多种 Model Context Protocol (MCP) 服务器，以演示该协议的功能和多样性。这些服务器使大型语言模型（LLMs）能够安全地访问工具和数据源。

## 参考实现

以下官方参考服务器展示了 MCP 核心功能及 SDK 的使用方法：

### 数据和文件系统
- **[Filesystem](https://github.com/modelcontextprotocol/servers/tree/main/src/filesystem)** - 提供具有可配置访问控制的安全文件操作
- **[PostgreSQL](https://github.com/modelcontextprotocol/servers/tree/main/src/postgres)** - 只读数据库访问，支持模式检查功能
- **[SQLite](https://github.com/modelcontextprotocol/servers/tree/main/src/sqlite)** - 数据库交互与业务智能功能
- **[Google Drive](https://github.com/modelcontextprotocol/servers/tree/main/src/gdrive)** - 支持 Google Drive 的文件访问及搜索功能

### 开发工具
- **[Git](https://github.com/modelcontextprotocol/servers/tree/main/src/git)** - 提供读取、搜索及操作 Git 仓库的工具
- **[GitHub](https://github.com/modelcontextprotocol/servers/tree/main/src/github)** - 支持仓库管理、文件操作以及与 GitHub API 的集成
- **[GitLab](https://github.com/modelcontextprotocol/servers/tree/main/src/gitlab)** - GitLab API 集成，支持项目管理
- **[Sentry](https://github.com/modelcontextprotocol/servers/tree/main/src/sentry)** - 访问并分析来自 Sentry.io 的问题数据

### 网络和浏览器自动化
- **[Brave Search](https://github.com/modelcontextprotocol/servers/tree/main/src/brave-search)** - 基于 Brave Search API 的网络及本地搜索功能
- **[Fetch](https://github.com/modelcontextprotocol/servers/tree/main/src/fetch)** - 优化用于 LLM 的网页内容获取与转换服务
- **[Puppeteer](https://github.com/modelcontextprotocol/servers/tree/main/src/puppeteer)** - 提供浏览器自动化与网页抓取功能

### 效率与通信
- **[Slack](https://github.com/modelcontextprotocol/servers/tree/main/src/slack)** - 管理频道及消息发送功能
- **[Google Maps](https://github.com/modelcontextprotocol/servers/tree/main/src/google-maps)** - 提供位置信息、路线及地点详细信息服务
- **[Memory](https://github.com/modelcontextprotocol/servers/tree/main/src/memory)** - 基于知识图谱的持久化记忆系统

### 人工智能及专用工具
- **[EverArt](https://github.com/modelcontextprotocol/servers/tree/main/src/everart)** - 使用多种模型进行 AI 图像生成
- **[Sequential Thinking](https://github.com/modelcontextprotocol/servers/tree/main/src/sequentialthinking)** - 通过思维流程实现动态问题解决
- **[AWS KB Retrieval](https://github.com/modelcontextprotocol/servers/tree/main/src/aws-kb-retrieval-server)** - 利用 Bedrock Agent Runtime 检索 AWS 知识库

## 官方集成

以下 MCP 服务器由其平台的公司维护：

- **[Axiom](https://github.com/axiomhq/mcp-server-axiom)** - 使用自然语言查询和分析日志、追踪及事件数据
- **[Browserbase](https://github.com/browserbase/mcp-server-browserbase)** - 云端自动化浏览器交互
- **[Cloudflare](https://github.com/cloudflare/mcp-server-cloudflare)** - 在 Cloudflare 开发平台上部署和管理资源
- **[E2B](https://github.com/e2b-dev/mcp-server)** - 在安全的云端沙盒中执行代码
- **[Neon](https://github.com/neondatabase/mcp-server-neon)** - 与 Neon 无服务器 Postgres 平台进行交互
- **[Obsidian Markdown Notes](https://github.com/calclavia/mcp-obsidian)** - 访问并搜索 Obsidian 笔记中的 Markdown 文件
- **[Qdrant](https://github.com/qdrant/mcp-server-qdrant/)** - 使用 Qdrant 向量搜索引擎实现语义内存功能
- **[Raygun](https://github.com/MindscapeHQ/mcp-server-raygun)** - 访问崩溃报告和监控数据
- **[Search1API](https://github.com/fatwang2/search1api-mcp)** - 提供搜索、抓取和站点地图的统一 API
- **[Stripe](https://github.com/stripe/agent-toolkit)** - 与 Stripe API 进行交互
- **[Tinybird](https://github.com/tinybirdco/mcp-tinybird)** - 与 Tinybird 无服务器 ClickHouse 平台交互

## 社区亮点

社区开发的服务器不断扩展 MCP 的功能：

- **[Docker](https://github.com/ckreiling/mcp-server-docker)** - 管理容器、镜像、卷及网络
- **[Kubernetes](https://github.com/Flux159/mcp-server-kubernetes)** - 管理 Pod、部署及服务
- **[Linear](https://github.com/jerhadf/linear-mcp-server)** - 项目管理及问题追踪
- **[Snowflake](https://github.com/datawiz168/mcp-snowflake-service)** - 与 Snowflake 数据库交互
- **[Spotify](https://github.com/varunneal/spotify-mcp)** - 控制 Spotify 播放及管理播放列表
- **[Todoist](https://github.com/abhiz123/todoist-mcp-server)** - 任务管理集成

> **注意：** 社区服务器未经测试，用户需自行承担使用风险。它们与 Anthropic 无关，也未得到其认可。

有关社区服务器的完整列表，请访问 [MCP Servers Repository](https://github.com/modelcontextprotocol/servers)。

## 开始使用

### 使用参考服务器

基于 TypeScript 的服务器可以直接使用 `npx`：

```bash
npx -y @modelcontextprotocol/server-memory
```

基于 Python 的服务器可以使用 `uvx`（推荐）或 `pip`：

```bash
# 使用 uvx
uvx mcp-server-git

# 使用 pip
pip install mcp-server-git
python -m mcp_server_git
```

### 配置 Claude

要在 Claude 中使用 MCP 服务器，将其添加到配置中：

```json
{
  "mcpServers": {
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/allowed/files"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "<YOUR_TOKEN>"
      }
    }
  }
}
```

## 其他资源

- [MCP Servers Repository](https://github.com/modelcontextprotocol/servers) - 参考实现和社区服务器的完整集合
- [Awesome MCP Servers](https://github.com/punkpeye/awesome-mcp-servers) - 精选的 MCP 服务器列表
- [MCP CLI](https://github.com/wong2/mcp-cli) - 用于测试 MCP 服务器的命令行工具
- [MCP Get](https://mcp-get.com) - 安装和管理 MCP 服务器的工具
- [Supergateway](https://github.com/supercorp-ai/supergateway) - 通过 SSE 运行 MCP stdio 服务器

访问我们的 [GitHub Discussions](https://github.com/orgs/modelcontextprotocol/discussions) 与 MCP 社区交流。