# Trading Bot Plugins

Official Codex and Claude Code plugin distribution for Trading Bot.

This repository is the source of truth for:

- the `trading-bot` plugin manifests;
- the `use-trading-bot` skill and protocol references;
- Codex and Claude Code marketplace catalogs;
- the remote MCP client configuration.

The MCP server implementation, Dashboard, OAuth server, Executor, and database
remain in the separately maintained Trading Bot service repository.

## Repository layout

```text
.agents/plugins/marketplace.json       # Codex marketplace
.claude-plugin/marketplace.json        # Claude Code marketplace
plugins/trading-bot/
  .codex-plugin/plugin.json            # Codex manifest
  .claude-plugin/plugin.json           # Claude Code manifest
  .mcp.json                            # shared MCP connection
  skills/use-trading-bot/              # shared skill and references
```

## Install with Codex

```bash
codex plugin marketplace add zaclau-dev/trading-bot-plugins --ref main
codex plugin add trading-bot@trading-bot-plugins
```

Start a new task after installation so Codex loads the plugin tools and skill.

## Install with Claude Code

Run these commands in a Claude Code session:

```text
/plugin marketplace add zaclau-dev/trading-bot-plugins
/plugin install trading-bot@trading-bot-plugins
```

Start a new session after installation.

## Local development

The committed MCP URL is `http://localhost:3000/mcp` for local integration
testing. Do not describe this repository as a production-ready public plugin
until `.mcp.json` and the Codex skill metadata point to the deployed HTTPS MCP
endpoint.

Validate both platforms before publishing:

```bash
python3 /path/to/plugin-creator/scripts/validate_plugin.py plugins/trading-bot
python3 /path/to/skill-creator/scripts/quick_validate.py \
  plugins/trading-bot/skills/use-trading-bot
claude plugin validate .
```

Deploy the MCP server first, update the endpoint and shared skill second, then
bump all plugin and marketplace versions together.
