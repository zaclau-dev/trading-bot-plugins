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

For a first-time installation, add the public marketplace before installing
the plugin:

```bash
codex plugin marketplace add zaclau-dev/trading-bot-plugins --ref main
codex plugin add trading-bot@trading-bot-plugins
```

Complete OAuth when prompted, then start a new task so Codex loads the Plugin,
Skill, and MCP tools.

If the marketplace and plugin are already installed, refresh the marketplace
and reinstall the current plugin version instead:

```bash
codex plugin marketplace upgrade trading-bot-plugins
codex plugin add trading-bot@trading-bot-plugins
```

The `upgrade` command is only for an already configured marketplace; it is not
a substitute for `marketplace add` on a new machine.

## Install with Claude Code

Run these commands in a Claude Code session:

```text
/plugin marketplace add zaclau-dev/trading-bot-plugins
/plugin install trading-bot@trading-bot-plugins
```

Start a new session after installation.

## Local development

The committed production MCP URL is
`https://trading-bot-five-pink.vercel.app/mcp`. Local integration testing may
temporarily override it in an uncommitted development configuration, but never
commit a loopback URL to a release.

Validate both platforms before publishing:

```bash
python3 /path/to/plugin-creator/scripts/validate_plugin.py plugins/trading-bot
python3 /path/to/skill-creator/scripts/quick_validate.py \
  plugins/trading-bot/skills/use-trading-bot
claude plugin validate .
```

Deploy the MCP server first, update the endpoint and shared skill second, then
bump all plugin and marketplace versions together.
