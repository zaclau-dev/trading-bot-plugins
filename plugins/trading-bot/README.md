# Trading Bot Plugin

Trading Bot ships one shared skill and MCP connection for Codex and Claude Code.

- Plugin: `trading-bot`
- Skill: `use-trading-bot`
- MCP server: `trading-bot`
- Marketplace: `trading-bot-plugins`

The default MCP endpoint is `http://localhost:3000/mcp` for local integration
testing. A production release must replace it with the deployed HTTPS endpoint
in both `.mcp.json` and `skills/use-trading-bot/agents/openai.yaml`.

## Install

Codex:

```bash
codex plugin marketplace add zaclau-dev/trading-bot-plugins --ref main
codex plugin add trading-bot@trading-bot-plugins
```

Claude Code:

```text
/plugin marketplace add zaclau-dev/trading-bot-plugins
/plugin install trading-bot@trading-bot-plugins
```

## OAuth and reconnection

The `/mcp` URL is a machine endpoint, not a login page. Let the MCP client
initiate OAuth and never copy access tokens into chat, configuration files, or
command arguments.

- In Codex, reconnect a confirmed expired or revoked grant with
  `codex mcp login trading-bot`.
- In Claude Code, open `/mcp`, select `trading-bot`, and complete
  authentication.
- A service restart alone does not revoke a stored OAuth grant. If tools are
  missing after the service returns, start a new task or session first.
