# Trading Bot Plugin

Trading Bot ships one shared skill and MCP connection for Codex and Claude Code.

- Plugin: `trading-bot`
- Skill: `use-trading-bot`
- MCP server: `trading-bot`
- Marketplace: `trading-bot-plugins`

The production MCP endpoint is
`https://trading-bot-v2-sepia.vercel.app/mcp`. It is shared by `.mcp.json`,
`skills/use-trading-bot/agents/openai.yaml`, and the connection-recovery steps
in `skills/use-trading-bot/SKILL.md`.

## Install and update

Codex first-time installation:

```bash
codex plugin marketplace add zaclau-dev/trading-bot-plugins --ref v2
codex plugin add trading-bot@trading-bot-plugins
```

Complete OAuth when prompted, then start a new Codex task so the Plugin, Skill,
and MCP tools are loaded.

Codex update for users who already installed the marketplace and plugin:

```bash
codex plugin marketplace upgrade trading-bot-plugins
codex plugin add trading-bot@trading-bot-plugins
```

Do not use `marketplace upgrade` for a first-time installation; the marketplace
must be added first.

Claude Code first-time installation:

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
