---
date: '2025-03-27T09:25:20+08:00'
draft: false
title: "示例客户端"
weight: 30
description: "支持 MCP 集成的应用程序列表"
---

本页面概述了支持 Model Context Protocol (MCP) 的应用程序。每个客户端可能支持不同的 MCP 功能，从而实现与 MCP 服务器的不同程度的集成。

## 功能支持矩阵

| 客户端                                      | [资源] | [提示] | [工具] | [采样] | Roots | 备注                                                                       |
|---------------------------------------------|--------|--------|--------|--------|-------|-----------------------------------------------------------------------------|
| [Claude Desktop App][Claude]                | ✅     | ✅     | ✅     | ❌     | ❌    | 全面支持所有 MCP 功能                                                      |
| [5ire][5ire]                                | ❌     | ❌     | ✅     | ❌     | ❌    | 支持工具                                                                    |
| [BeeAI Framework][BeeAI Framework]          | ❌     | ❌     | ✅     | ❌     | ❌    | 在代理工作流中支持工具                                                     |
| [Cline][Cline]                              | ✅     | ❌     | ✅     | ❌     | ❌    | 支持工具和资源                                                              |
| [Continue][Continue]                        | ✅     | ✅     | ✅     | ❌     | ❌    | 全面支持所有 MCP 功能                                                      |
| [Cursor][Cursor]                            | ❌     | ❌     | ✅     | ❌     | ❌    | 支持工具                                                                    |
| [Emacs Mcp][Mcp.el]                         | ❌     | ❌     | ✅     | ❌     | ❌    | 在 Emacs 中支持工具                                                        |
| [Firebase Genkit][Genkit]                   | ⚠️     | ✅     | ✅     | ❌     | ❌    | 支持通过工具执行资源列表查询                                               |
| [GenAIScript][GenAIScript]                  | ❌     | ❌     | ✅     | ❌     | ❌    | 支持工具                                                                    |
| [Goose][Goose]                              | ❌     | ❌     | ✅     | ❌     | ❌    | 支持工具                                                                    |
| [LibreChat][LibreChat]                      | ❌     | ❌     | ✅     | ❌     | ❌    | 支持代理的工具                                                             |
| [mcp-agent][mcp-agent]                      | ❌     | ❌     | ✅     | ⚠️     | ❌    | 支持工具，服务器连接管理和代理工作流                                        |
| [Roo Code][Roo Code]                        | ✅     | ❌     | ✅     | ❌     | ❌    | 支持工具和资源                                                              |
| [Sourcegraph Cody][Cody]                    | ✅     | ❌     | ❌     | ❌     | ❌    | 通过 OpenCTX 支持资源                                                       |
| [Superinterface][Superinterface]            | ❌     | ❌     | ✅     | ❌     | ❌    | 支持工具                                                                    |
| [TheiaAI/TheiaIDE][TheiaAI/TheiaIDE]        | ❌     | ❌     | ✅     | ❌     | ❌    | 在 Theia AI 和 Theia IDE 中支持代理使用工具                                 |
| [Windsurf Editor][Windsurf]                 | ❌     | ❌     | ✅     | ❌     | ❌    | 使用 AI Flow 支持协作开发的工具                                            |
| [Zed][Zed]                                  | ❌     | ✅     | ❌     | ❌     | ❌    | 提示以斜杠命令的形式出现                                                   |
| [OpenSumi][OpenSumi]                        | ❌     | ❌     | ✅     | ❌     | ❌    | 在 OpenSumi 中支持工具                                                     |

[Claude]: https://claude.ai/download
[Cursor]: https://cursor.com
[Zed]: https://zed.dev
[Cody]: https://sourcegraph.com/cody
[Genkit]: https://github.com/firebase/genkit
[Continue]: https://github.com/continuedev/continue
[GenAIScript]: https://microsoft.github.io/genaiscript/reference/scripts/mcp-tools/
[Cline]: https://github.com/cline/cline
[LibreChat]: https://github.com/danny-avila/LibreChat
[TheiaAI/TheiaIDE]: https://eclipsesource.com/blogs/2024/12/19/theia-ide-and-theia-ai-support-mcp/
[Superinterface]: https://superinterface.ai
[5ire]: https://github.com/nanbingxyz/5ire
[BeeAI Framework]: https://i-am-bee.github.io/beeai-framework
[mcp-agent]: https://github.com/lastmile-ai/mcp-agent
[Mcp.el]: https://github.com/lizqwerscott/mcp.el
[Roo Code]: https://roocode.com
[Goose]: https://block.github.io/goose/docs/goose-architecture/#interoperability-with-extensions
[Windsurf]: https://codeium.com/windsurf

[资源]: https://modelcontextprotocol.io/docs/concepts/resources
[提示]: https://modelcontextprotocol.io/docs/concepts/prompts
[工具]: https://modelcontextprotocol.io/docs/concepts/tools
[采样]: https://modelcontextprotocol.io/docs/concepts/sampling

## 客户端详情

### Claude Desktop App

Claude 桌面应用程序全面支持 MCP，可以与本地工具和数据源进行深度集成。

**主要功能：**

- 全面支持资源，允许附加本地文件和数据
- 支持提示模板
- 工具集成以执行命令和脚本
- 本地服务器连接以增强隐私和安全性

> ⓘ 注意：Claude.ai 网页应用程序目前不支持 MCP，仅在桌面应用程序中支持 MCP 功能。

### 5ire

[5ire](https://github.com/nanbingxyz/5ire) 是一个开源的跨平台桌面 AI 助手，支持通过 MCP 服务器使用工具。

**主要功能：**

- 内置的 MCP 服务器可快速启用/禁用。
- 用户可以通过修改配置文件添加更多服务器。
- 开源且用户友好，适合初学者。
- 未来计划不断改进对 MCP 的支持。

### BeeAI Framework

[BeeAI Framework](https://i-am-bee.github.io/beeai-framework) 是一个开源框架，用于构建、部署和大规模服务强大的代理工作流。该框架包含 **MCP Tool**，这是一个原生功能，可简化 MCP 服务器在代理工作流中的集成。

**主要功能：**

- 无缝将 MCP 工具集成到代理工作流中。
- 从已连接的 MCP 客户端快速实例化框架原生工具。
- 计划未来支持代理的 MCP 功能。

**了解更多：**

- [在代理工作流中使用 MCP 工具的示例](https://i-am-bee.github.io/beeai-framework/#/typescript/tools?id=using-the-mcptool-class)

### Cline

[Cline](https://github.com/cline/cline) 是 VS Code 中的一个自动化编程代理，可以根据用户许可编辑文件、运行命令、使用浏览器等。

**主要功能：**

- 通过自然语言创建和添加工具（例如“添加一个搜索网络的工具”）。
- 在 `~/Documents/Cline/MCP` 目录中共享 Cline 创建的自定义 MCP 服务器。
- 显示配置的 MCP 服务器及其工具、资源和任何错误日志。

### Continue

[Continue](https://github.com/continuedev/continue) 是一个开源的 AI 代码助手，内置对所有 MCP 功能的支持。

**主要功能：**

- 输入 "@" 来引用 MCP 资源。
- 提示模板以斜杠命令呈现。
- 在聊天中直接使用内置和 MCP 工具。
- 支持 VS Code 和 JetBrains IDE，兼容任何 LLM。

### Cursor
[Cursor](https://docs.cursor.com/advanced/model-context-protocol) 是一款 AI 代码编辑器。

**主要功能：**

- 在 Cursor Composer 中支持 MCP 工具。
- 支持 STDIO 和 SSE。

### Emacs Mcp

[Emacs Mcp](https://github.com/lizqwerscott/mcp.el) 是一个为 Emacs 设计的客户端，用于与 MCP 服务器连接和交互。它为 Emacs 的 AI 插件（如 [gptel](https://github.com/karthink/gptel) 和 [llm](https://github.com/ahyatt/llm)）提供 MCP 工具支持，增强了 Emacs 生态系统中 AI 工具的功能。

**主要功能：**
- 为 Emacs 提供 MCP 工具支持。

### Firebase Genkit

[Genkit](https://github.com/firebase/genkit) 是 Firebase 的开发者工具包，用于在应用程序中构建和集成 GenAI 功能。[genkitx-mcp](https://github.com/firebase/genkit/tree/main/js/plugins/mcp) 插件允许以客户端身份使用 MCP 服务器，或从 Genkit 的工具和提示创建 MCP 服务器。

**主要功能：**

- 客户端支持工具和提示（部分支持资源）。
- 在 Genkit 的开发者界面中支持丰富的功能发现。
- 与 Genkit 的现有工具和提示无缝互操作。
- 支持多种顶级提供商的 GenAI 模型。

### GenAIScript

使用 [GenAIScript](https://microsoft.github.io/genaiscript/)（基于 JavaScript）以编程方式为 LLM 组装提示，编排 LLM、工具和数据。

**主要功能：**

- JavaScript 工具箱，便于操作提示。
- 抽象化设计使开发高效便捷。
- 无缝集成 Visual Studio Code。

### Goose

[Goose](https://github.com/block/goose) 是一个开源的 AI 代理，旨在通过自动化编程任务提升您的软件开发效率。

**主要功能：**
- 通过工具向 Goose 暴露 MCP 功能。

- MCP 可直接通过 [扩展目录](https://block.github.io/goose/v1/extensions/)、CLI 或 UI 安装。
- Goose 支持 [构建自定义 MCP 服务器](https://block.github.io/goose/docs/tutorials/custom-extensions)，以扩展其功能。
- 内置开发工具、网络抓取、自动化、内存功能以及与 JetBrains 和 Google Drive 的集成。

### LibreChat

[LibreChat](https://github.com/danny-avila/LibreChat) 是一个开源、可定制的 AI 聊天 UI，支持多个 AI 提供商，包括 MCP 集成。

**主要功能：**

- 通过 MCP 服务器扩展现有工具生态系统，包括 [代码解释器](https://www.librechat.ai/docs/features/code_interpreter) 和图像生成工具。
- 使用顶级提供商的多种 LLM 向可定制的 [代理](https://www.librechat.ai/docs/features/agents) 添加工具。
- 开源且可自托管，支持安全的多用户环境。
- 未来计划包括扩展对 MCP 功能的支持。

### mcp-agent

[mcp-agent] 是一个简单且可组合的框架，用于通过 Model Context Protocol 构建代理。

**主要功能：**

- 自动管理 MCP 服务器连接。
- 向 LLM 暴露多个服务器的工具。
- 实现由 [构建有效代理](https://www.anthropic.com/research/building-effective-agents) 定义的每种模式。
- 支持暂停/恢复工作流信号，例如等待人类反馈的功能。

### Roo Code

[Roo Code](https://roocode.com) 通过 MCP 实现 AI 编程辅助。

**主要功能：**

- 支持 MCP 工具和资源。
- 与开发工作流集成。
- 可扩展的 AI 功能。

### Sourcegraph Cody

[Cody](https://openctx.org/docs/providers/modelcontextprotocol) 是 Sourcegraph 的 AI 编程助手，通过 OpenCTX 实现 MCP。

**主要功能：**

- 支持 MCP 资源。
- 与 Sourcegraph 的代码智能集成。
- 使用 OpenCTX 作为抽象层。
- 计划未来支持更多 MCP 功能。

### Superinterface

[Superinterface](https://superinterface.ai) 是一个 AI 基础设施和开发者平台，用于构建具有 MCP 支持的应用内 AI 助手，以及交互式组件、客户端函数调用等。

**主要功能：**

- 在通过 React 组件或脚本标签嵌入的助手中使用 MCP 服务器的工具。
- 支持 SSE 传输。
- 支持任意 AI 提供商（如 OpenAI、Anthropic、Ollama 等）的模型。

### TheiaAI/TheiaIDE

[Theia AI](https://eclipsesource.com/blogs/2024/10/07/introducing-theia-ai/) 是一个用于构建增强 AI 功能工具和 IDE 的框架。[AI 驱动的 Theia IDE](https://eclipsesource.com/blogs/2024/10/08/introducting-ai-theia-ide/) 是基于 Theia AI 构建的开放灵活的开发环境。

**主要功能：**

- **工具集成**: Theia AI 允许 AI 代理（包括 Theia IDE 中的代理）使用 MCP 服务器进行无缝工具交互。
- **可定制提示**: Theia IDE 允许用户定义和调整提示，动态集成 MCP 服务器以支持定制的工作流。
- **自定义代理**: Theia IDE 支持创建利用 MCP 功能的自定义代理，允许用户即时设计专用工作流。

Theia AI 和 Theia IDE 的 MCP 集成为用户提供了灵活性，使它们成为探索和适应 MCP 的强大平台。

**了解更多：**

- [Theia IDE 和 Theia AI MCP 公告](https://eclipsesource.com/blogs/2024/12/19/theia-ide-and-theia-ai-support-mcp/)
- [下载 AI 支持的 Theia IDE](https://theia-ide.org/)

### Windsurf Editor

[Windsurf Editor](https://codeium.com/windsurf) 是一个结合 AI 辅助和开发者工作流的代理型 IDE。它具有创新的 AI Flow 系统，支持协作和独立的 AI 交互，同时保持开发者的完全控制。

**主要功能：**

- 革命性的 AI Flow 协作范式。
- 智能代码生成和理解。
- 丰富的开发工具，支持多种模型。

### Zed

[Zed](https://zed.dev/docs/assistant/model-context-protocol) 是一个高性能代码编辑器，内置 MCP 支持，专注于提示模板和工具集成。

**主要功能：**

- 提示模板以斜杠命令的形式在编辑器中显示。
- 工具集成以增强编码工作流。
- 与编辑器功能及工作空间上下文紧密集成。
- 暂不支持 MCP 资源。

### OpenSumi

[OpenSumi](https://github.com/opensumi/core) 是一个帮助快速构建 AI 本地化 IDE 产品的框架。

**主要功能：**

- 在 OpenSumi 中支持 MCP 工具。
- 支持内置 IDE MCP 服务器和自定义 MCP 服务器。

## 将 MCP 支持添加到您的应用程序

如果您已将 MCP 支持添加到您的应用程序，欢迎提交拉取请求以将其添加到此列表中。MCP 集成可以为您的用户提供强大的上下文 AI 功能，并使您的应用程序成为不断增长的 MCP 生态系统的一部分。

添加 MCP 支持的好处：

- 允许用户带入自己的上下文和工具
- 加入不断扩展的可互操作 AI 应用生态系统
- 为用户提供灵活的集成选项
- 支持以本地为优先的 AI 工作流

要开始在您的应用中实现 MCP，请参考我们的 [Python](https://github.com/modelcontextprotocol/python-sdk) 或 [TypeScript SDK 文档](https://github.com/modelcontextprotocol/typescript-sdk)。

## 更新与纠正

此列表由社区维护。如果您发现任何错误或希望更新有关您的应用程序中 MCP 支持的信息，请提交拉取请求或 [在我们的文档仓库开一个问题](https://github.com/modelcontextprotocol/docs/issues)。
