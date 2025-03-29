---
date: '2025-03-28T09:16:28+08:00'
draft: false
title: ' Model Context Protocol 规范（2025-03-26）'
cascade:
  - type: "docs"
---

> **协议修订**: 2025-03-26

[Model Context Protocol](https://modelcontextprotocol.io) (MCP) 是一个开放协议，
为 LLM 应用与外部数据源和工具之间的无缝集成提供了可能。无论您是构建一个 AI 驱动的 IDE，增强聊天界面，还是创建自定义 AI 工作流，MCP 提供了一种标准化方式，将 LLM 与所需的上下文连接起来。

本规范定义了基于 [schema.ts](https://github.com/modelcontextprotocol/specification/blob/main/schema/draft/schema.ts) 中的 TypeScript 模式的权威协议要求。

要获取实现指南和示例，请访问
[modelcontextprotocol.io](https://modelcontextprotocol.io)。

本文档中的关键词 "MUST"、"MUST NOT"、"REQUIRED"、"SHALL"、"SHALL NOT"、"SHOULD"、"SHOULD NOT"、"RECOMMENDED"、"NOT RECOMMENDED"、"MAY" 和 "OPTIONAL" 的解释方式与 [BCP 14](https://datatracker.ietf.org/doc/html/bcp14)
[[RFC2119](https://datatracker.ietf.org/doc/html/rfc2119)]
[[RFC8174](https://datatracker.ietf.org/doc/html/rfc8174)] 中的描述一致，仅当它们全部以大写形式出现时适用。

## 概述

MCP 提供了一种标准化方式，帮助应用程序：

- 与语言模型共享上下文信息
- 向 AI 系统公开工具和功能
- 构建可组合的集成与工作流

该协议使用 [JSON-RPC](https://www.jsonrpc.org/) 2.0 消息来建立以下对象之间的通信：

- **Hosts**: 发起连接的 LLM 应用
- **Clients**: 在 host 应用中运行的连接器
- **Servers**: 提供上下文和功能的服务

MCP 从 [Language Server Protocol](https://microsoft.github.io/language-server-protocol/) 中获得了一些灵感，后者通过一个标准化方式，为整个开发工具生态系统添加编程语言支持。类似地，MCP 标准化了将额外上下文和工具集成到 AI 应用生态中的方式。

## 关键细节

### 基础协议

- [JSON-RPC](https://www.jsonrpc.org/) 消息格式
- 有状态的连接
- 服务端和客户端的功能协商

### 功能

服务端可以向客户端提供以下任意功能：

- **Resources**: 上下文和数据，供用户或 AI 模型使用
- **Prompts**: 为用户提供的模板化消息和工作流
- **Tools**: AI 模型可执行的功能

客户端可以向服务端提供以下功能：

- **Sampling**: 服务端发起的代理行为和递归的 LLM 交互

### 额外工具

- 配置
- 进度跟踪
- 取消操作
- 错误报告
- 日志记录

## 安全与信任 & 安全性

Model Context Protocol 通过任意数据访问和代码执行路径提供了强大的功能。与这种能力相伴而来的是所有实现者必须认真考虑的重要安全性和信任问题。

### 关键原则

1. **用户同意与控制**

   - 用户必须明确同意并理解所有数据访问和操作
   - 用户必须保留对共享哪些数据和进行哪些操作的控制权
   - 实现者应提供明确的用户界面，以供用户审查和授权活动

2. **数据隐私**

   - 在向服务端暴露用户数据之前，Host 必须获得用户的明确同意
   - 在未经用户同意的情况下，Host 不得将资源数据传输到其他地方
   - 应通过适当的访问控制保护用户数据

3. **工具安全性**

   - 工具代表任意代码执行，必须谨慎对待。
     - 特别是，工具行为（例如注解）的描述应被视为不可信，除非来自可信服务端。
   - Host 必须获得用户的明确同意后才能调用任何工具
   - 用户应在授权使用某工具前了解其功能

4. **LLM 采样控制**
   - 用户必须明确批准任何 LLM 采样请求
   - 用户应能够控制：
     - 是否允许采样
     - 将要发送的具体 Prompt
     - 服务端能够看到的结果
   - 协议有意限制服务端对 Prompt 的可见性

### 实施指南

虽然 MCP 本身无法在协议层面实施这些安全原则，但实现者 **应当**：

1. 在其应用程序中构建稳健的同意与授权流程
2. 提供与安全性相关的清晰文档
3. 实施适当的访问控制和数据保护措施
4. 在其集成中遵循安全最佳实践
5. 在设计功能时充分考虑隐私问题

## 了解更多

探索每个协议组件的详细规范：

- **[基础协议](basic)**  
- **[服务端功能](server)**  
- **[客户端功能](client)**  
- **[贡献指南](contributing)**
