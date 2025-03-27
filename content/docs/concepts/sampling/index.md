---
date: '2025-03-27T11:04:02+08:00'
draft: false
title: "采样"
description: "让您的服务器通过客户端向 LLM 请求补全"
---

采样是一项强大的 MCP 功能。它允许服务器通过客户端向 LLM 请求补全，从而实现复杂的智能行为，同时保持安全性和隐私性。

> MCP 的这一功能尚未在 Claude Desktop 客户端中支持。

## 采样如何工作

采样流程遵循以下步骤：

1. 服务器向客户端发送 `sampling/createMessage` 请求
2. 客户端审查请求并可以修改它
3. 客户端从 LLM 中进行采样
4. 客户端审查生成的补全内容
5. 客户端将结果返回给服务器

这种“人工-参与式”设计确保用户对 LLM 所能看到的内容和生成的内容保持控制权。

## 消息格式

采样请求使用标准化的消息格式：

```typescript
{
  messages: [
    {
      role: "user" | "assistant",
      content: {
        type: "text" | "image",

        // 对于文本内容：
        text?: string,

        // 对于图像内容：
        data?: string,             // Base64 编码
        mimeType?: string
      }
    }
  ],
  modelPreferences?: {
    hints?: [{
      name?: string                // 模型名称/家族的建议
    }],
    costPriority?: number,         // 0-1，降低成本的重要性
    speedPriority?: number,        // 0-1，低延迟的重要性
    intelligencePriority?: number  // 0-1，功能强大的重要性
  },
  systemPrompt?: string,
  includeContext?: "none" | "thisServer" | "allServers",
  temperature?: number,
  maxTokens: number,
  stopSequences?: string[],
  metadata?: Record<string, unknown>
}
```

## 请求参数

### Messages

`messages` 数组包含发送到 LLM 的对话历史记录。每条消息包括：

- `role`: "user"（用户）或 "assistant"（助手）
- `content`: 消息内容，可以是：
  - 包含 `text` 字段的文本内容
  - 包含 `data`（Base64 编码）和 `mimeType` 字段的图像内容

### Model preferences

`modelPreferences` 对象允许服务器指定模型选择偏好：

- `hints`: 一个模型名称建议的数组，客户端可用其选择合适的模型：
  - `name`: 可以匹配完整或部分模型名称的字符串（如 "claude-3", "sonnet"）
  - 客户端可能将建议映射为来自不同提供商的等效模型
  - 多个建议按优先顺序进行评估
- 优先值范围 (0-1)：
  - `costPriority`: 降低成本的重要性
  - `speedPriority`: 响应低延迟的重要性
  - `intelligencePriority`: 模型高级功能的重要性

客户端根据这些偏好及其可用模型做最终模型选择。

### System prompt

可选的 `systemPrompt` 字段允许服务器请求特定的系统提示词。客户端可能修改或忽略此内容。

### Context inclusion

`includeContext` 参数指定要包含的 MCP 上下文范围：

- `"none"`: 不包含额外上下文
- `"thisServer"`: 包括来自请求服务器的上下文
- `"allServers"`: 包括来自所有连接 MCP 服务器的上下文

客户端控制实际包含的上下文。

### 采样参数

通过以下参数微调 LLM 的采样：

- `temperature`: 控制生成内容的随机性 (0.0 到 1.0)
- `maxTokens`: 生成的最大 Token 数
- `stopSequences`: 结束生成的序列数组
- `metadata`: 提供商特定的额外参数

## 响应格式

客户端返回的补全结果格式如下：

```typescript
{
  model: string,  // 使用的模型名称
  stopReason?: "endTurn" | "stopSequence" | "maxTokens" | string,
  role: "user" | "assistant",
  content: {
    type: "text" | "image",
    text?: string,
    data?: string,
    mimeType?: string
  }
}
```

## 示例请求

以下是向客户端请求采样的示例：

```json
{
  "method": "sampling/createMessage",
  "params": {
    "messages": [
      {
        "role": "user",
        "content": {
          "type": "text",
          "text": "当前目录中的文件有哪些？"
        }
      }
    ],
    "systemPrompt": "你是一位友好的文件系统助手。",
    "includeContext": "thisServer",
    "maxTokens": 100
  }
}
```

## 最佳实践

实施采样时：

1. 始终提供清晰、结构良好的提示
2. 适当地处理文本和图像内容
3. 设置合理的 Token 限制
4. 使用 `includeContext` 包含相关上下文
5. 在使用响应之前验证其内容
6. 优雅地处理错误
7. 考虑对采样请求的速率限制
8. 文档化预期的采样行为
9. 使用不同的模型参数进行测试
10. 监控采样成本

## 人工监督控制

采样设计具有人工监督功能：

### 对提示词的监督

- 客户端应向用户展示建议的提示词
- 用户应该能够修改或拒绝提示词
- 系统提示词可以被过滤或修改
- 上下文包含由客户端控制

### 对补全的监督

- 客户端应向用户展示补全内容
- 用户应该能够修改或拒绝补全内容
- 客户端可以过滤或修改补全内容
- 用户控制所使用的模型

## 安全考虑

实施采样时需注意：

- 验证所有消息内容
- 清理敏感信息
- 实施适当的速率限制
- 监控采样的使用情况
- 对传输中的数据进行加密
- 处理用户数据的隐私问题
- 审核采样请求
- 控制成本风险
- 实施超时机制
- 优雅地处理模型错误

## 常见模式

### 智能工作流

采样可实现以下智能模式：

- 读取并分析资源
- 基于上下文做决策
- 生成结构化数据
- 处理多步骤任务
- 提供交互式帮助

### 上下文管理

上下文处理的最佳实践：

- 请求最小必要的上下文
- 清晰地组织上下文
- 处理上下文大小限制
- 根据需要更新上下文
- 清理过时的上下文

### 错误处理

健壮的错误处理包括：

- 捕获采样失败
- 处理超时错误
- 管理速率限制
- 验证响应内容
- 提供备用行为
- 适当记录错误

## 限制

需了解以下限制：

- 采样依赖于客户端能力
- 用户控制采样行为
- 上下文大小有限制
- 可能有速率限制
- 需考虑成本
- 模型的可用性可能不同
- 响应时间可能有变动
- 并非所有内容类型都被支持
