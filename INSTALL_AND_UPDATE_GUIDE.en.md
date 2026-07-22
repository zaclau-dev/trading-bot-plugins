# Trading Bot Installation and Update Guide

[简体中文](INSTALL_AND_UPDATE_GUIDE.zh-CN.md)

## Integration methods

| Platform | Integration |
| --- | --- |
| Codex | Plugin, including the Skill and MCP configuration |
| Claude Code | Plugin, including the Skill and MCP configuration |
| Other agent platforms | MCP, with an optional standalone Skill |

MCP Tool Schemas are loaded automatically when the client connects. No separate installation is required.

## Installation

### Codex

For a first-time installation on a new machine, run both commands in order:

```bash
codex plugin marketplace add zaclau-dev/trading-bot-plugins --ref main
codex plugin add trading-bot@trading-bot-plugins
```

Complete OAuth authorization when prompted, then start a new Codex task so the Plugin, Skill, and MCP tools are loaded. Do not use `marketplace upgrade` for a first-time installation because the marketplace has not been configured yet.

### Claude Code

```bash
claude plugin marketplace add zaclau-dev/trading-bot-plugins
claude plugin install trading-bot@trading-bot-plugins
```

Restart Claude Code or start a new session. If authorization is required, open `/mcp` and select `trading-bot`.

### Other agent platforms

Prefer the platform's official MCP command. If it has no MCP command, add this in its MCP settings:

```json
{
  "mcpServers": {
    "trading-bot": {
      "type": "http",
      "url": "https://trading-bot-five-pink.vercel.app/mcp"
    }
  }
}
```

Note: `type: "http"` identifies the MCP transport. The actual connection is
encrypted because the endpoint URL uses `https://`; do not change the type to
`"https"`.

Use the official Trading Bot HTTPS MCP endpoint shown above. Save the configuration, restart the agent or reconnect MCP, and complete OAuth authorization when prompted.

Install a standalone Skill only when the official instructions support it. There is no universal one-command installer across agent platforms; use an official platform-specific command.

## Updates

### Codex

Use these commands only when the `trading-bot-plugins` marketplace and `trading-bot` plugin are already installed:

```bash
codex plugin marketplace upgrade trading-bot-plugins
codex plugin add trading-bot@trading-bot-plugins
```

Start a new Codex task after updating. Run this only if authorization has expired:

```bash
codex mcp login trading-bot
```

### Claude Code

```bash
claude plugin marketplace update trading-bot-plugins
claude plugin update trading-bot@trading-bot-plugins
```

Restart Claude Code or start a new session. If authorization has expired, open `/mcp`, select `trading-bot`, and authorize again.

### Other agent platforms

- Compatible MCP server update: no installation is required; start a new session or reconnect MCP.
- MCP endpoint or authentication change: use the official platform-specific command or update the MCP configuration manually.
- Standalone Skill update: replace the Skill through the platform, then restart or start a new session.

## Verification

Send this in a new task or session:

```text
List the projects and actions currently supported by Trading Bot. Do not execute any transaction.
```

If the Trading Bot tools do not appear, check the Plugin installation, MCP endpoint, OAuth authorization, and network access to the MCP service.

> Never put access tokens, private keys, or wallet signing keys in chat or configuration files.
