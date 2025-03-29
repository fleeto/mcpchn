---
title: 授权
weight: 15
draft: false
---

## 1. 简介

### 1.1 目的和范围

模型上下文协议在传输层提供授权功能，使 MCP 客户端能够代表资源所有者向受限的 MCP 服务器发出请求。此规范定义了基于 HTTP 的传输的授权流程。

### 1.2 协议需求

对于 MCP 实现，授权是可选的。支持时：

- 使用基于 HTTP 的传输的实现应该符合此规范。
- 使用 STDIO 传输的实现不应遵循此规范，而是从环境中检索凭据。
- 使用替代传输的实现必须遵循其协议既定的最佳实践。

### 1.3 合规性

该授权机制基于下面列出的既定规范，但实现了其部分选定的功能子集，以确保安全性和互操作性，同时保持简单性：

- [OAuth 2.1 IETF DRAFT  OAuth 2.1 IETF 草案](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-v2-1-12)
- OAuth 2.0 Authorization Server Metadata（[RFC8414](https://datatracker.ietf.org/doc/html/rfc8414)）
- OAuth 2.0 Dynamic Client Registration Protocol（[RFC7591](https://datatracker.ietf.org/doc/html/rfc7591)）
- OAuth 2.0 动态客户端注册协议（[RFC7591](https://datatracker.ietf.org/doc/html/rfc7591)）

## 2 授权流程

### 2.1 概述

1. MCP 身份验证实现必须为机密客户端和公共客户端实施 OAuth 2.1 并采取适当的安全措施。
1. MCP 身份验证实现应该支持 OAuth 2.0 动态客户端注册协议（[RFC7591](https://datatracker.ietf.org/doc/html/rfc7591)）。
1. MCP 服务器应该实现，MCP 客户端必须实现 OAuth 2.0 授权服务器元数据 ( RFC8414 )。不支持授权服务器元数据的服务器必须遵循默认的 URI 架构。

### 2.2 基本 OAuth 2.1 授权

当需要授权但客户端尚未证明时，服务器必须以 `HTTP 401 Unauthorized` 进行响应。

客户端发起 [OAuth 2.1 IETF 草案](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-v2-1-12) 收到 `HTTP 401 Unauthorized` 后的授权流程。

下面演示了使用 PKCE 的公共客户端的基本 OAuth 2.1。

```mermaid
sequenceDiagram
    participant B as User-Agent (Browser)
    participant C as Client
    participant M as MCP Server

    C->>M: MCP Request
    M->>C: HTTP 401 Unauthorized
    Note over C: Generate code_verifier and code_challenge
    C->>B: Open browser with authorization URL + code_challenge
    B->>M: GET /authorize
    Note over M: User logs in and authorizes
    M->>B: Redirect to callback URL with auth code
    B->>C: Callback with authorization code
    C->>M: Token Request with code + code_verifier
    M->>C: Access Token (+ Refresh Token)
    C->>M: MCP Request with Access Token
    Note over C,M: Begin standard MCP message exchange
```

### 2.3 服务器元数据发现

对于服务器功能发现：

- MCP 客户端必须遵循 [RFC8414](https://datatracker.ietf.org/doc/html/rfc8414) 中定义的 OAuth 2.0 授权服务器元数据协议。
- MCP 服务器应该遵循 OAuth 2.0 授权服务器元数据协议。
- 不支持 OAuth 2.0 授权服务器元数据协议的 MCP 服务器， 必须支持后备 URL。

发现流程如下所示：

```mermaid
sequenceDiagram
    participant C as Client
    participant S as Server

    C->>S: GET /.well-known/oauth-authorization-server
    alt Discovery Success
        S->>C: 200 OK + Metadata Document
        Note over C: Use endpoints from metadata
    else Discovery Failed
        S->>C: 404 Not Found
        Note over C: Fall back to default endpoints
    end
    Note over C: Continue with authorization flow
```

#### 2.3.1 服务器元数据发现标头

MCP 客户端应在服务器元数据发现期间包含标头 `MCP-Protocol-Version: <protocol-version>` ，以允许 MCP 服务器根据 MCP 协议版本做出响应。

例如：`MCP-Protocol-Version: 2024-11-05`

#### 2.3.2 授权基准 URL

必须通过丢弃任何现有 `path` 组件，根据 MCP 服务器 URL 确定授权基准 URL。例如：

如果 MCP 服务器 URL 为 `https://api.example.com/v1/mcp` ，则：

- 授权基础 URL 为 `https://api.example.com`
- 元数据端点必须位于 `https://api.example.com/.well-known/oauth-authorization-server`

这可确保授权端点始终位于托管 MCP 服务器的域的根级别，而不管 MCP 服务器 URL 中的任何路径组件如何。

#### 2.3.3 没有元数据发现的服务器的回退

对于未实现 OAuth 2.0 授权服务器元数据的服务器，客户端 必须使用相对于授权基准 URL（如第 2.3.2 节所定义）的以下默认端点路径：

| 端点 | 默认路径 | 描述 |
| --- | --- | --- |
| 授权端点 | `/authorize` | 用于授权请求 |
| Token 端点 | `/token` | 用于 Token 交换和刷新 |
| 注册端点  | `/register` | 用于注册动态客户端 |

例如，对于托管在 `https://api.example.com/v1/mcp` 的 MCP 服务器，默认端点将是：

- `https://api.example.com/authorize`
- `https://api.example.com/token`
- `https://api.example.com/register`

客户端必须先尝试通过元数据文档发现端点，然后再返回默认路径。使用默认路径时，所有其他协议要求保持不变。

### 2.4 注册动态客户端

MCP 客户端和服务器应该支持 [OAuth 2.0 动态客户端注册协议](https://datatracker.ietf.org/doc/html/rfc7591) 允许 MCP 客户端无需用户交互即可获取 OAuth 客户端 ID。这提供了 客户端自动注册新服务器的标准化方式，这是至关重要的 对于 MCP，因为：

- 客户端无法提前知道所有可能的服务器
- 手动注册会给用户带来不便
- 实现与新服务器的无缝连接
- 服务器可以实现自己的注册策略

任何不支持动态客户端注册的 MCP 服务器都需要提供其他方法来获取客户端 ID（以及客户端密钥，如果适用）。对于这些服务器之一，MCP 客户端必须执行以下任一操作：

- 为该 MCP 服务器专门编写一个客户端 ID（以及客户端密钥，如果适用），或者
- 在用户自己注册 OAuth 客户端后（例如，通过服务器托管的配置界面），向用户呈现一个 UI，允许他们输入这些详细信息。

### 2.5 授权流程步骤

完整授权流程如下

```mermaid
sequenceDiagram
    participant B as User-Agent (Browser)
    participant C as Client
    participant M as MCP Server

    C->>M: GET /.well-known/oauth-authorization-server
    alt Server Supports Discovery
        M->>C: Authorization Server Metadata
    else No Discovery
        M->>C: 404 (Use default endpoints)
    end

    alt Dynamic Client Registration
        C->>M: POST /register
        M->>C: Client Credentials
    end

    Note over C: Generate PKCE Parameters
    C->>B: Open browser with authorization URL + code_challenge
    B->>M: Authorization Request
    Note over M: User /authorizes
    M->>B: Redirect to callback with authorization code
    B->>C: Authorization code callback
    C->>M: Token Request + code_verifier
    M->>C: Access Token (+ Refresh Token)
    C->>M: API Requests with Access Token
```

### 2.5.1 决策流程概述

```mermaid
flowchart TD
    A[Start Auth Flow] --> B{Check Metadata Discovery}
    B -->|Available| C[Use Metadata Endpoints]
    B -->|Not Available| D[Use Default Endpoints]

    C --> G{Check Registration Endpoint}
    D --> G

    G -->|Available| H[Perform Dynamic Registration]
    G -->|Not Available| I[Alternative Registration Required]

    H --> J[Start OAuth Flow]
    I --> J

    J --> K[Generate PKCE Parameters]
    K --> L[Request Authorization]
    L --> M[User Authorization]
    M --> N[Exchange Code for Tokens]
    N --> O[Use Access Token]
```

### 2.6 Access Token 的用法

#### 2.6.1 Token 的前置条件

1. MCP 客户端必须使用授权请求标头字段 第 5.1.1 节 ：`Authorization: Bearer <access-token>`

    请注意，从客户端到服务器的每个 HTTP 请求都必须包含授权，即使它们是同一逻辑会话的一部分。

1. Access Token 不得包含在 URI 查询字符串中

请求示例：

```plaintext
GET /v1/contexts HTTP/1.1
Host: mcp.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

#### 2.6.2 Token 处理

资源服务器必须按照以下说明验证访问令牌 第 5.2 节 。如果验证失败，服务器必须按照 第 5.3 节 错误处理要求。无效或过期的令牌必须收到 HTTP 401 响应。

### 2.7 安全注意事项

必须实施以下安全要求：

1. 客户端必须按照 OAuth 2.0 最佳实践安全地存储令牌
1. 服务器应该强制执行令牌过期和轮换
1. 所有授权端点必须通过 HTTPS 提供服务
1. 服务器必须验证重定向 URI，以防止开放重定向漏洞
1. 重定向 URI 必须是本地主机 URL 或 HTTPS URL

### 2.8 错误处理

服务器必须针对授权错误返回适当的 HTTP 状态代码：

| Status Code | Description  | Usage |
| --- | --- | --- |
| 401         | 未授权 | 需要授权或令牌无效 |
| 403         | 禁止    | 范围无效或权限不足 |
| 400         | 错误的请求 | 授权请求格式错误 |

### 2.9 实现方法的要求

1. 必须遵循 OAuth 2.1 安全最佳实践
1. 所有客户端都需要 PKCE
1. 令牌轮换以增强安全性
1. 令牌生命周期应该根据安全要求进行限制

### 2.10 第三方授权流程

#### 2.10.1 概述

MCP 服务器可以支持通过第三方授权服务器进行委托授权。在此流程中，MCP 服务器既充当 OAuth 客户端（对第三方授权服务器而言），又充当 OAuth 授权服务器（对 MCP 客户端而言）。

#### 2.10.2 流程描述

第三方授权流程包括以下步骤：

1. MCP 客户端向 MCP 服务器发起标准 OAuth 流程
1. MCP 服务器将用户重定向至第三方授权服务器
1. 用户向第三方服务器授权
1. 第三方服务器使用授权码重定向回 MCP 服务器
1. MCP 服务器交换代码以获取第三方访问令牌
1. MCP 服务器生成自己的访问令牌并绑定到第三方会话
1. MCP 服务器与 MCP 客户端完成原始 OAuth 流程

```mermaid
sequenceDiagram
    participant B as User-Agent (Browser)
    participant C as MCP Client
    participant M as MCP Server
    participant T as Third-Party Auth Server

    C->>M: Initial OAuth Request
    M->>B: Redirect to Third-Party /authorize
    B->>T: Authorization Request
    Note over T: User authorizes
    T->>B: Redirect to MCP Server callback
    B->>M: Authorization code
    M->>T: Exchange code for token
    T->>M: Third-party access token
    Note over M: Generate bound MCP token
    M->>B: Redirect to MCP Client callback
    B->>C: MCP authorization code
    C->>M: Exchange code for token
    M->>C: MCP access token
```

#### 2.10.3 绑定会话的需求

实现第三方授权的 MCP 服务器必须 ：

1. 维护第三方 Token 和已发行的 MCP Token 之间的安全映射
1. 在兑换 MCP Token 之前验证第三方 Token 状态
1. 实施适当的 Token 生命周期管理
1. 处理第三方 Token 过期和续订

#### 2.10.4 安全注意事项

在实现第三方授权时，服务器必须 ：

1. 验证所有重定向 URI
1. 安全存储第三方凭证
1. 实施适当的会话超时处理
1. 考虑令牌链的安全影响
1. 对第三方身份验证失败实施适当的错误处理

## 3 最佳实践

### 3.1 本地客户端作为公共 OAuth 2.1 客户端

我们强烈建议本地客户端将 OAuth 2.1 作为公共客户端实现：

1. 利用 PKCE 进行授权请求，以防止拦截攻击
1. 选择合适的方式安全地存储 Token
1. 会话过程中遵照最佳实践刷新 Token
1. 正确处理 Token 过期和续订

### 3.2 授权元数据发现

我们强烈建议所有客户端都实现元数据发现。这样可以减少用户手动提供端点或客户端回退到定义的默认值的需要。

### 3.3 动态客户端注册

由于客户端事先不知道 MCP 服务器的集合，我们强烈建议实现动态客户端注册。这允许应用程序自动向 MCP 服务器注册，无需用户手动获取客户端 ID。