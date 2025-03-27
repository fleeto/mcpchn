---
date: '2025-03-27T09:39:33+08:00'
draft: false
title: "使用 LLM 构建 MCP"
description: "使用 Claude 或者其它 LLM 加速您的 MCP 开发！"
---
本指南将帮助您利用 LLM 来构建自定义的模型上下文协议 (MCP) 服务器和客户端。我们将在本教程中聚焦于 Claude，但您也可以使用任何前沿的 LLM。

## 准备文档

在开始之前，收集必要的文档以帮助 Claude 理解 MCP：

1. 访问 https://modelcontextprotocol.io/llms-full.txt 并复制完整的文档内容
2. 前往 [MCP TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk) 或 [Python SDK 仓库](https://github.com/modelcontextprotocol/python-sdk)
3. 复制 README 文件及其他相关文档
4. 将这些文件粘贴到与 Claude 的对话中

## 描述您的服务器

提供完文档后，清楚地向 Claude 描述您想要构建的服务器类型。具体描述以下内容：

- 服务器将暴露哪些资源
- 将提供哪些工具
- 应包含哪些提示
- 需要与哪些外部系统交互

例如：
```
构建一个 MCP 服务器，它：
- 连接到我公司的 PostgreSQL 数据库
- 将表的模式作为资源暴露
- 提供运行只读 SQL 查询的工具
- 包括针对常见数据分析任务的提示
```

## 与 Claude 一起工作

在用 Claude 构建 MCP 服务器时：

1. 从核心功能开始，然后逐步添加更多功能
2. 请 Claude 解释任何您不理解的代码部分
3. 根据需要请求修改或改进
4. 让 Claude 帮助您测试服务器并处理边界情况

Claude 可以帮助实现所有关键的 MCP 功能：

- 资源管理和暴露
- 工具定义和实现
- 提示模板和处理器
- 错误处理和日志记录
- 连接与传输设置

## 最佳实践

使用 Claude 构建 MCP 服务器时：

- 将复杂的服务器拆分为更小的部分
- 在继续下一步之前，彻底测试每个组件
- 考虑安全性 - 验证输入并适当地限制访问
- 为将来的维护做好充分的代码文档
- 严格遵循 MCP 协议规范

## 下一步

在 Claude 帮助您构建好服务器后：

1. 仔细审查生成的代码
2. 使用 MCP Inspector 工具测试服务器
3. 将服务器连接到 Claude.app 或其他 MCP 客户端
4. 根据实际使用和反馈进行迭代

请记住，随着需求的变化，Claude 可以帮助您修改和改进服务器。

需要更多指导？只需向 Claude 提出关于实现 MCP 功能或解决出现的问题的具体问题。
