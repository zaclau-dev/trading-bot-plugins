# Trading Bot 安装与更新指南

[English](INSTALL_AND_UPDATE_GUIDE.en.md)

## 接入方式

| 平台 | 接入方式 |
| --- | --- |
| Codex | Plugin（包含 Skill 和 MCP 配置） |
| Claude Code | Plugin（包含 Skill 和 MCP 配置） |
| 其他 Agent 平台 | MCP，可选独立 Skill |

MCP Tool Schema 由客户端连接时自动读取，无需单独安装。

## 安装

### Codex

外部新用户或新机器首次安装时，按顺序执行以下两条命令：

```bash
codex plugin marketplace add zaclau-dev/trading-bot-plugins --ref v2
codex plugin add trading-bot@trading-bot-plugins
```

按提示完成 OAuth 授权，然后新建 Codex 任务，让 Plugin、Skill 和 MCP 工具生效。首次安装不能使用 `marketplace upgrade`，因为新用户尚未添加该 marketplace。

### Claude Code

```bash
claude plugin marketplace add zaclau-dev/trading-bot-plugins
claude plugin install trading-bot@trading-bot-plugins
```

安装后重启 Claude Code 或新建会话。需要授权时，打开 `/mcp` 并选择 `trading-bot`。

### 其他 Agent 平台

优先使用平台官方 MCP 命令。平台没有 MCP 命令时，在其 MCP 设置中添加：

```json
{
  "mcpServers": {
    "trading-bot": {
      "type": "http",
      "url": "https://trading-bot-v2-sepia.vercel.app/mcp"
    }
  }
}
```

注意：这里的 `type: "http"` 是 MCP 客户端使用的传输类型；实际连接地址是
`https://`，因此连接仍然通过 HTTPS 加密。不要把 `type` 改成 `"https"`。

使用上方 Trading Bot 官方 HTTPS MCP 地址。保存后重启 Agent 或重新连接 MCP，并按提示完成 OAuth 授权。

只有官方说明明确支持时，才需要另外安装独立 Skill。不同 Agent 平台没有通用的一键安装命令，请使用官方提供的平台专用命令。

## 更新

### Codex

只有已经安装过 `trading-bot-plugins` marketplace 和 `trading-bot` plugin 的用户才执行：

```bash
codex plugin marketplace upgrade trading-bot-plugins
codex plugin add trading-bot@trading-bot-plugins
```

更新后新建 Codex 任务。使用插件时，如果需要登录授权且当前环境支持执行命令，助手会主动打开授权页面，用户只需在页面完成登录和授权。无法自动发起时，可在插件的连接设置中授权，或在终端执行：

```bash
codex mcp login trading-bot
```

### Claude Code

```bash
claude plugin marketplace update trading-bot-plugins
claude plugin update trading-bot@trading-bot-plugins
```

更新后重启 Claude Code 或新建会话。如果尚未登录授权或授权已失效，打开 `/mcp`，选择 `trading-bot` 并完成登录授权。

### 其他 Agent 平台

- MCP 服务端兼容更新：无需安装；新建会话或重新连接 MCP 即可。
- MCP 地址或认证方式变化：使用官方平台专用命令，或手动更新 MCP 配置。
- 独立 Skill 更新：按平台方式替换 Skill，然后重启或新建会话。

## 验证

在新任务或会话中发送：

```text
请列出 Trading Bot 当前支持的项目和操作，不要执行任何交易。
```

Codex 需要授权时，助手会通过客户端主动打开登录授权页面。页面显示完成后，仍需等待客户端确认登录成功，才算连接完成。自动发起不可用时，才需要在插件的连接设置中授权。登录成功后若工具仍不可用，先使用客户端支持的连接刷新；没有刷新入口时再新建会话。登录服务繁忙时，请按提示稍后重试。

> 不要在聊天或配置文件中填写访问令牌、私钥或钱包签名密钥。
