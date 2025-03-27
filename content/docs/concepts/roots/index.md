---
date: '2025-03-27T10:12:08+08:00'
draft: false
title: "Roots（根）"
description: "理解 MCP 中的根"
---

Roots 是 MCP 中定义服务器可操作边界的一个概念。它们为客户端提供了一种方式，能够向服务器告知相关资源及其位置。

## 什么是 Roots？

根（Root）是一个客户端建议服务器关注的 URI。当客户端连接到服务器时，它声明服务器应该处理哪些根。尽管根主要用于文件系统路径，但它们可以是任何有效的 URI，包括 HTTP URLs。

例如，根可能是：

```plaintext
file:///home/user/projects/myapp
https://api.example.com/v1
```

## 为什么要使用 Roots？

Roots 有以下几个重要用途：

1. **指导**：它们告知服务器有关的资源和位置。
2. **清晰**：Roots 使哪些资源属于工作空间变得清晰。
3. **组织性**：多个 Roots 让你能够同时处理不同的资源。

## Roots 如何工作

当一个客户端支持 Roots 时，它会：

1. 在连接时声明 `roots` 能力。
2. 向服务器提供建议的 Roots 列表。
3. （如果支持）当 Roots 发生变化时，通知服务器。

尽管 Roots 是信息性的，并不具有严格的约束力，但服务器应当：

1. 尊重提供的 Roots。
2. 使用 Root URI 定位并访问资源。
3. 优先处理根边界内的操作。

## 常见用例

Roots 通常用于定义：

- 项目目录
- 代码库位置
- API 端点
- 配置文件位置
- 资源边界

## 最佳实践

在使用 Roots 时：

1. 仅建议必要的资源。
2. 为 Roots 使用清晰、描述性强的名称。
3. 监控根的可访问性。
4. 优雅地处理根的变化。

## 示例

以下是一个典型的 MCP 客户端如何暴露 Roots 的示例：

```json
{
  "roots": [
    {
      "uri": "file:///home/user/projects/frontend",
      "name": "Frontend Repository"
    },
    {
      "uri": "https://api.example.com/v1",
      "name": "API Endpoint"
    }
  ]
}
```

这个配置建议服务器同时关注一个本地代码库和一个 API 端点，并将它们逻辑上分隔。
