# Repository instructions

- This repository is the source of truth for the Trading Bot Codex and Claude
  Code plugin package, marketplace catalogs, shared skill, and MCP client
  configuration.
- Keep the Codex and Claude manifests platform-specific, but keep
  `skills/use-trading-bot`, its references, and `.mcp.json` shared.
- Keep the MCP server implementation, Dashboard, OAuth server, Executor, and
  database in the separate Trading Bot service repository; do not duplicate
  that runtime here.
- Validate the Codex plugin, shared skill, and Claude marketplace before every
  release. Bump the manifest and marketplace versions together.
- When the sibling service changes any externally visible adapter action,
  parameter, supported asset, adapter version, preview field, confirmation
  behavior, or safety rule, synchronize the shared Skill/reference and bump all
  plugin and marketplace versions in the same task. Do not treat the runtime
  implementation as complete until the synchronized plugin validates.
- Deploy and verify a compatible MCP server before publishing skill or endpoint
  changes.
