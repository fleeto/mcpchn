---
date: '2025-03-27T15:09:03+08:00'
draft: false
title: 路线图
description: 我们对 Model Context Protocol（2025上半年）的发展计划
---

Model Context Protocol 正在迅速发展。本页面概述了我们对 **2025年上半年**关键优先事项和未来方向的当前想法，尽管随着项目发展，这些可能会有显著变化。

> 此处展示的想法并非承诺——我们可能会以不同于描述的方式解决这些挑战，或者有些挑战可能根本不会实现。与此同时，这也不是一个_详尽_的列表；我们可能会纳入这里未提及的工作。

我们鼓励社区参与！每个部分都链接到相关讨论，您可以了解更多信息并贡献您的想法。

## 远程 MCP 支持

我们的首要任务是实现[远程 MCP 连接](https://github.com/modelcontextprotocol/specification/discussions/102)，使客户端能够通过互联网安全连接至 MCP 服务器。关键举措包括：

- [**认证与授权**](https://github.com/modelcontextprotocol/specification/discussions/64)：添加标准认证功能，特别是专注于 OAuth 2.0 的支持。

- [**服务发现**](https://github.com/modelcontextprotocol/specification/discussions/69)：定义客户端如何发现并连接到远程 MCP 服务器。

- [**无状态操作**](https://github.com/modelcontextprotocol/specification/discussions/102)：探讨 MCP 是否可以涵盖无服务器环境的需求，在这些环境中操作需要主要保持无状态。

## 参考实现

为了帮助开发者使用 MCP，我们计划提供以下文档：

- **客户端示例**：全面的参考客户端实现，展示所有协议功能。
- **协议起草**：为提议和纳入新协议功能提供精简流程。

## 分发与发现

展望未来，我们正在探索让 MCP 服务器更容易访问的方法。我们可能研究的领域包括：

- **包管理**：MCP 服务器的标准化打包格式。
- **安装工具**：简化 MCP 客户端上的服务器安装过程。
- **沙箱化**：通过服务器隔离来提高安全性。
- **服务器注册表**：提供可发现的 MCP 服务器公共目录。

## 代理支持

我们正在扩展 MCP 对[复杂代理工作流](https://github.com/modelcontextprotocol/specification/discussions/111)的支持，特别是关注以下方面：

- [**分层代理系统**](https://github.com/modelcontextprotocol/specification/discussions/94)：通过命名空间和拓扑感知，改进对代理树的支持。

- [**交互式工作流**](https://github.com/modelcontextprotocol/specification/issues/97)：更好地处理用户权限和信息请求，支持跨代理层级的操作，并为用户（而非模型）提供输出的方式。

- [**流式结果**](https://github.com/modelcontextprotocol/specification/issues/117)：提供长期运行代理操作的实时更新。

## 更广泛的生态系统

我们也致力于以下方向：

- **社区主导的标准开发**：通过平等参与和共享治理，建立一个协作生态系统，让所有 AI 提供商都能帮助将 MCP 打造成开放标准，确保其满足多样化的 AI 应用和使用场景需求。
- [**附加模态**](https://github.com/modelcontextprotocol/specification/discussions/88)：突破文本，支持音频、视频及其他格式。
- **标准化**：考虑通过标准化机构进行标准化。

## 加入我们

我们欢迎社区参与，共同塑造 MCP 的未来。访问我们的 [GitHub Discussions](https://github.com/orgs/modelcontextprotocol/discussions)来加入讨论并贡献您的想法。
