---
date: '2025-03-27T14:35:51+08:00'
draft: lse
title: 调试
description: Model Context Protocol (MCP) 集成调试的全面指南
---

有效的调试是开发 MCP 服务器或将其与应用程序集成时的关键。本指南涵盖了 MCP 生态系统中可用的调试工具和方法。

> 本指南适用于 macOS。适用于其他平台的指南即将推出。

## 调试工具概览

MCP 提供了多个不同层次的调试工具：

1. **MCP Inspector**
   - 交互式调试界面
   - 直接测试服务器
   - 查看 [Inspector 指南](/docs/tools/inspector) 了解详情

2. **Claude 桌面开发工具**
   - 集成测试
   - 日志收集
   - 与 Chrome DevTools 集成

3. **服务器日志**
   - 自定义日志实现
   - 错误追踪
   - 性能监控

## 在 Claude 桌面版中调试

### 检查服务器状态

Claude.app 界面提供基本的服务器状态信息：

1. 点击连接图标查看：
   - 已连接的服务器
   - 可用的提示和资源

2. 点击锤子图标查看：
   - 模型可用的工具

### 查看日志

从 Claude 桌面查看详细的 MCP 日志：

```bash
# 实时跟踪日志
tail -n 20 -F ~/Library/Logs/Claude/mcp*.log
```

日志捕获内容包括：

- 服务器连接事件
- 配置问题
- 运行时错误
- 消息交换

### 使用 Chrome DevTools

从 Claude 桌面访问 Chrome 的开发工具以排查客户端错误：

1. 创建一个 `developer_settings.json` 文件，并将 `allowDevTools` 设置为 true：

    ```bash
    echo '{"allowDevTools": true}' > ~/Library/Application\ Support/Claude/developer_settings.json
    ```

2. 打开 DevTools：`Command-Option-Shift-i`

注意：将会看到两个 DevTools 窗口：

- 主内容窗口
- 应用标题栏窗口

使用 Console 面板检查客户端错误。

使用 Network 面板检查：

- 消息负载
- 连接时序

## 常见问题

### 工作目录

在 Claude 桌面版中使用 MCP 服务器时：

- 通过 `claude_desktop_config.json` 启动的服务器，其工作目录可能未定义（如 macOS 上的 `/`），因为 Claude 桌面版可以从任意位置启动。
- 始终使用绝对路径在您的配置和 `.env` 文件中，以确保可靠的操作。
- 直接通过命令行测试服务器时，其工作目录将是您运行命令的位置。

例如，在 `claude_desktop_config.json` 中，使用：

```json
{
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/username/data"]
}
```

而不是相对路径如 `./data`

### 环境变量

MCP 服务器仅会自动继承一部分环境变量，如 `USER`、`HOME` 和 `PATH`。

如果需要覆盖默认变量或提供自定义变量，可以在 `claude_desktop_config.json` 中指定 `env` 键：

```json
{
  "myserver": {
    "command": "mcp-server-myapp",
    "env": {
      "MYAPP_API_KEY": "some_key",
    }
  }
}
```

### 服务器初始化

常见的初始化问题：

1. **路径问题**
   - 服务器可执行文件路径不正确
   - 缺少必需文件
   - 权限问题
   - 尝试为 `command` 使用绝对路径

2. **配置错误**
   - JSON 语法无效
   - 缺少必需字段
   - 类型不匹配

3. **环境问题**
   - 缺少环境变量
   - 变量值错误
   - 权限限制

### 连接问题

当服务器连接失败时：

1. 检查 Claude 桌面版的日志
2. 验证服务器进程是否正在运行
3. 使用 [Inspector](/docs/tools/inspector) 独立测试
4. 验证协议兼容性

## 实现日志记录

### 服务器端日志记录

构建使用本地 stdio [传输](/docs/concepts/transports) 的服务器时，记录到 stderr（标准错误输出）的所有消息将自动被主机应用程序（如 Claude 桌面）捕获。

> 本地 MCP 服务器不应将消息记录到 stdout（标准输出），否则会干扰协议操作。

对于所有 [传输](/docs/concepts/transports)，还可以通过发送日志消息通知向客户端提供日志记录：

{{< tabpane persist="lang" text=true >}}
{{% tab header="Python" %}}

```python
server.request_context.session.send_log_message(
  level="info",
  data="Server started successfully",
)
```

{{% /tab %}}
{{% tab header="TypeScript" %}}

```typescript
server.sendLoggingMessage({
  level: "info",
  data: "Server started successfully",
});
```

{{% /tab %}}
{{% /tabpane %}}

重要的日志事件包括：

- 初始化步骤
- 资源访问
- 工具执行
- 错误状态
- 性能指标

### 客户端日志记录

在客户端应用程序中：

1. 启用调试日志
2. 监视网络流量
3. 跟踪消息交换
4. 记录错误状态

## 调试工作流程

### 开发周期

1. 初始开发
   - 使用 [Inspector](/docs/tools/inspector) 进行基本测试
   - 实现核心功能
   - 添加日志记录点

2. 集成测试
   - 在 Claude 桌面版中测试
   - 监控日志
   - 检查错误处理

### 测试更改

为了高效测试更改：

- **配置更改**：重启 Claude 桌面版
- **服务器代码更改**：使用 Command-R 重新加载
- **快速迭代**：在开发过程中使用 [Inspector](/docs/tools/inspector)

## 最佳实践

### 日志记录策略

1. **结构化日志**
   - 使用一致的格式
   - 包含上下文
   - 添加时间戳
   - 跟踪请求 ID

2. **错误处理**
   - 记录堆栈跟踪
   - 包含错误上下文
   - 跟踪错误模式
   - 监控恢复情况

3. **性能跟踪**
   - 记录操作耗时
   - 监控资源使用
   - 跟踪消息大小
   - 测量延迟

### 安全注意事项

调试时需注意：

1. **敏感数据**
   - 清理日志
   - 保护凭证
   - 屏蔽个人信息

2. **访问控制**
   - 验证权限
   - 检查认证
   - 监控访问模式

## 寻求帮助

遇到问题时：

1. **第一步**
   - 检查服务器日志
   - 使用 [Inspector](/docs/tools/inspector) 测试
   - 审查配置
   - 验证环境

2. **支持渠道**
   - 提交 GitHub issue
   - 参与 GitHub 讨论

3. **提供信息**
   - 日志摘录
   - 配置文件
   - 复现步骤
   - 环境详情

## 后续步骤

- **[MCP Inspector](/docs/tools/inspector)**：学习使用 MCP Inspector
