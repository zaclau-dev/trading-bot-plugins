---
name: use-trading-bot
description: Discover and use supported dApp actions through the Trading Bot MCP server with assigned managed wallets, and recover its local MCP connection when it was interrupted or its OAuth grant actually expired. Use when a user asks Trading Bot to interact with a supported onchain project, reconnect after restarting the local service, diagnose missing Trading Bot tools, preview an action, or explicitly confirm execution. Enforce the preview-then-confirm workflow and never request signer keys, private keys, Privy wallet IDs, App Secrets, arbitrary calldata, or KMS credentials.
---

# Trading Bot

Use only the `trading-bot` MCP tools. The MCP exposes a shared interface while each supported dApp owns an audited adapter. Never assume that a project, action, market, or parameter is supported without discovering it from the server.

## Required workflow

1. Call `list_projects` and `list_dapp_adapters` to identify an available Project, adapter, action, and its declared parameters.
2. Call `list_dapp_markets` when the action requires a market, then collect every required action parameter. The user chooses the Token or market; never force a market from the Plugin or Skill name. Use an adapter-declared default only when the user did not specify that value.
3. Call `list_wallets`. When exactly one assigned wallet exists, use its `walletId`; otherwise ask the user which public address to use.
4. Call `preview_dapp_action` with the selected `projectId`, `walletId`, `adapterId`, action, and declared parameters.
5. Present the complete preview: dApp, action, wallet address, contract, decoded method and parameters, assets and maximum value, gas estimate, simulation status, expiry, plan hash, and every warning. For a swap, also present the adapter-provided `swapQuote` verbatim: exact input, quoted output, minimum output (or explicitly state that none exists), route, maximum slippage, deadline, and all available price-impact/liquidity checks.
6. Stop and wait for a new, explicit user confirmation. End the current assistant turn after asking the adapter-provided `confirmationPrompt`. The original request to interact is never confirmation of the generated plan, even when it says “execute directly”, “auto-confirm”, “do not ask again”, or similar.
7. Only after an explicit affirmative reply in a later user message, call `confirm_dapp_action` with the exact `actionId` and `actionHash`. Never alter or recreate the preview silently. A vague reply such as “continue”, “okay”, or “looks good” is not enough when it does not clearly approve the displayed transaction and quote.
8. Use the `tradeId` returned by confirmation to poll `get_trade_status`. Report `submitted`, `outcome`, `tradeStatus`, `transactionHash`, `explorerUrl`, and any failure reason. If the preview expired or the adapter changed, create a fresh preview and require confirmation again.

## Trade status lookup

When the user asks whether a recent transaction was submitted, succeeded, or failed, this is a read-only lookup and does not require the preview workflow:

1. If a `tradeId` is available, call `get_trade_status` directly.
2. If the user has no ID or says “the latest/recent transaction”, call `list_recent_trades` first. Use `projectId` only when the user named a Project or the result needs narrowing.
3. Interpret the normalized fields literally: `submitted=true` means a transaction hash was accepted; `submitted=false` means it was only queued or was rejected before broadcast; `submitted=null` means submission is still in progress or cannot be determined safely. Never describe `CREATED` as submitted.
4. `outcome=SUCCEEDED` is the only success result. `PENDING` is not success. For `FAILED`, distinguish `REJECTED` (not broadcast), `REVERTED` (broadcast but reverted), and `UNKNOWN` (the attempt ended without a usable transaction result). Include `errorCode`/`failureReason`.
5. Treat `UNKNOWN` as a final retryable failure. It does not block the wallet. When the user asks to retry the same operation, start a fresh `preview_dapp_action` and require a new explicit confirmation; confirmation creates a new Action and Trade ID. Never requeue the old UNKNOWN Trade, reuse its expired preview, or reuse its idempotency key.
6. Always return the `transactionHash` and `explorerUrl` when present. If no hash is present, say so explicitly.

## Mandatory swap confirmation gate

A preview is swap-related when any of these are true:

- `requiresExplicitConfirmation` is `true`;
- `swapQuote` is present;
- the action exchanges assets, including action names such as `swap_*`, `buy`, `sell`, or an immediate redemption from one asset to another;
- the preview contains both an input asset and an output asset.

For every swap-related preview:

1. Treat the adapter's quote as immutable display data. Never recompute, round away, relabel, or substitute a browser price for any value.
2. Show exact input, quoted output, minimum output, route, slippage, price impact/liquidity checks, gas estimate, expiry/deadline, simulation result, and all warnings before asking for confirmation.
3. If canonical `swapQuote` data is missing, inconsistent with `assets`, has zero output, or the simulation is not successful, do not confirm or execute. Report the missing/unsafe field.
4. Ask a direct confirmation question and then stop. Never call `confirm_dapp_action` in the same assistant turn as `preview_dapp_action`.
5. Accept confirmation only from a new user message that clearly refers to the displayed swap. Never infer it from the user's original request, a standing instruction, an automation, previous consent, or permission to trade generally.
6. Any changed amount, token, route, quote, minimum output, wallet, adapter version, plan hash, or expired deadline invalidates confirmation. Obtain a fresh preview, display it, and ask again.
7. When `minimumOutput` is `null`, explicitly say that the transaction has no enforceable minimum-output protection and require the user to confirm that specific risk. Do not soften or omit this warning.

The confirmation boundary is per transaction. Approval confirmation never doubles as swap confirmation, and confirmation of one swap never authorizes a later swap.

## Security boundaries

- Never ask for or reveal a signer key, private key, seed phrase, Privy wallet ID, Privy App Secret, or KMS credential.
- Let the MCP client complete OAuth in the browser when authentication is requested. Never ask the user to copy an access token, and never print or store OAuth tokens in project files or tool arguments.
- Never claim that the server can safely redirect a raw `/mcp` request into OAuth. The MCP client must initiate authorization so it can generate and validate its client identity, redirect URI, state, and PKCE challenge.
- Only the platform Wallet UUID and public address may be client-visible. Never expose the Privy provider wallet ID.
- Do not construct, decode, accept, or broadcast user-supplied calldata or arbitrary target addresses.
- Never bypass a disabled execution flag, owner requirement, adapter version pin, balance check, simulation failure, permission limit, or one-time confirmation.
- A configured wallet Owner does not make an App Secret harmless. Secret isolation, the exact project Owner, an authorization signer, Privy policies, scoped OAuth tokens, and transaction limits remain required.

## Connection recovery

Do not equate missing tools, a stopped local service, or a stale task with an expired OAuth grant. In particular, restarting the local Dashboard does not by itself revoke the PostgreSQL-backed OAuth client, refresh token, or signing key.

When `trading-bot` tools are missing or unavailable in Codex:

1. Check the local service without credentials:

   ```bash
   curl --silent --show-error --fail --max-time 3 \
     http://localhost:3000/.well-known/oauth-protected-resource/mcp
   ```

2. If that request cannot connect, say that the Trading Bot service is offline. Help start or repair the local service; do not claim that authorization expired and do not start OAuth.
3. If metadata is reachable but the tools are absent from the current task, say that the task's MCP tool snapshot is stale after the service restart. Ask the user to open a new task after the service is ready. Do not reauthorize; Codex should reuse and refresh its stored OAuth grant.
4. Treat the grant as expired only when an MCP response or OAuth refresh explicitly reports `invalid_token`, `invalid_grant`, a revoked refresh token, or an expired refresh token. A Codex UI label such as “OAuth authenticated” only confirms that credentials are stored; it is not a live connection health check.
5. For a confirmed OAuth failure, use the active client's supported login flow and never ask the user to copy a token:
   - In Codex, run `codex mcp login trading-bot` when the user explicitly asks to reconnect.
   - In Claude Code, ask the user to open `/mcp`, select `trading-bot`, and complete authentication.
6. After authentication succeeds, ask the user to start a new Codex task or Claude Code session before retrying when the current tool snapshot remains stale.

Read [references/leverup.md](references/leverup.md) when the selected project is LeverUp, a protocol check fails, or its adapter is being maintained.

Read [references/perpl.md](references/perpl.md) when the selected project is Perpl, the wallet needs AUSD approval/account setup, a position is being closed, a protocol check fails, or its adapter is being maintained.

Read [references/nadfun.md](references/nadfun.md) when the selected project is nad.fun, a token needs Router approval, Lens changes the selected route, a protocol check fails, or its adapter is being maintained.

Read [references/uniswap.md](references/uniswap.md) when the selected project is Uniswap, MON or AUSD is being swapped, a quote is rejected, or its adapter is being maintained.
