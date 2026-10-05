---
name: use-trading-bot
description: Discover and use supported dApp actions through the configured Trading Bot MCP server with assigned managed wallets, handle exact token approvals without an extra user confirmation, and recover its connection when interrupted or its OAuth grant actually expired. Use for capability and wallet discovery, action previews, approval recovery, confirmation, Execution status, and connection diagnosis. Keep swap confirmation separate and never request signer keys, private keys, Privy wallet IDs, App Secrets, opaque calldata, or KMS credentials.
---

# Trading Bot

For normal discovery, preview, confirmation, and status operations, use only the `trading-bot` MCP tools. The MCP exposes a shared interface while each supported dApp owns an audited adapter. The client login commands in **Connection recovery** are the only exception.

Treat live MCP responses as the source of truth. Use this precedence order: live adapter capability and preview, then the selected protocol reference, then examples in this Skill. Never assume that a project, action, market, parameter, default, or adapter version is supported without discovering it from the server. If a reference conflicts with a live capability or preview, stop and report a possible client/server version mismatch; never guess which value to use.

Use this Skill as the authority for confirmation sources and assistant-turn timing. Exact token approvals follow **Exact token approval authorization** below. Other actions require either a later user confirmation or an applicable active Goal. Protocol references may require separate actions, fresh previews, or additional checks, but must follow these confirmation sources.

## UTC and confirmation-time authority

- Treat every MCP timestamp as a UTC instant. A trailing `Z` means UTC; a different calendar date in the task's local timezone does not make the timestamp stale or expired.
- For a fresh preview, use only the server-derived `isExpired` and `remainingSeconds`. Do not compare timestamp text, calendar dates, the task's `current_date`, the device clock, or any client-local timezone to decide expiry.
- On a later confirmation message, use `get_dapp_action` when a current check is needed and trust its server-derived Action `status`. If the status is `PREVIEWED`, call `confirm_dapp_action` with the exact stored `actionId` and `actionHash`; do not pre-reject it using any client clock.
- The MCP server's `confirm_dapp_action` result is the final expiry authority. Only a server `ACTION_EXPIRED` response or server-derived `EXPIRED` status proves expiry. Never claim that the Trading Bot clock is stale merely because its UTC date differs from the local date.
- Localize a UTC timestamp only for display, while preserving the original UTC value. A display conversion must never affect whether the action is confirmed.

## Route the request

Classify the user's request before calling tools:

- **Read-only discovery**: call only the tools needed to list the requested projects, adapters, markets, wallets, or positions. Do not create a preview unless the user asks to prepare an action or a live preview is required to assess whether that exact action is currently executable.
- **Prepare an action**: follow **Preview and execution workflow** below. Every action begins with a preview. An exact token approval may be confirmed in the same turn under **Exact token approval authorization**. Other actions require a later user confirmation or **Goal-delegated confirmation**; “execute directly” does not bypass the swap confirmation gate.
- **Confirm a displayed preview**: use only the exact preview already shown and an applicable authorization: **Exact token approval authorization**, **Goal-delegated confirmation**, or an affirmative reply in a later user message. When exactly one unexpired preview is awaiting confirmation, interpret a brief conversational affirmative in context instead of requiring the user to repeat the transaction details.
- **Look up an Execution**: follow **Execution status lookup**. Do not restart discovery or create a preview merely to report status.
- **Recover the connection**: follow **Connection recovery** without treating missing tools as proof that OAuth expired.

## Goal-delegated confirmation

An active persisted Codex Goal is a valid delegated confirmation source when its objective explicitly instructs the agent to auto-confirm transaction previews, proceed without per-transaction confirmation questions, make and confirm the user's in-scope decisions, or equivalent. Treat this as an alternative to a later user confirmation message, not as a reason to skip previewing or validating the transaction.

When Goal-delegated confirmation applies:

- Use it only for wallets, Projects, dApps, objectives, amounts or budgets, deadlines, and failure handling stated or bounded by the active Goal. When the Goal explicitly delegates action and parameter decisions, select only from live adapter capabilities, markets, owned positions, declared defaults, and current wallet state. Follow the Goal's optimization objective; when it asks to minimize cost or loss and gives no amount, use the smallest live executable amount consistent with completing the requested interaction.
- Present the complete preview required by this Skill, then call `confirm_dapp_action` with that preview's exact `actionId` and `actionHash` in the same assistant turn. Do not ask the user to confirm and do not stop merely to obtain another user message.
- Reuse the delegation independently for each later in-scope preview while that Goal remains active. Every transaction still requires its own fresh preview, exact hash, server validity checks, and `confirm_dapp_action` call; the Goal does not turn one plan hash into approval for another.
- Interpret `requiresExplicitConfirmation: true` as requiring the complete preview and a distinct `confirm_dapp_action` call. It does not require a later user message when Goal-delegated confirmation applies.
- Stop using the delegation when the Goal is paused, edited so the authorization is removed, cleared, or completed. Also stop and ask the user when an action would expand beyond the Goal's stated scope or limits, or when the Goal leaves a genuinely necessary choice unresolved and no conservative in-scope choice exists.
- Never use Goal delegation to override an expired preview, failed simulation, missing or inconsistent swap data, disabled execution flag, adapter/version mismatch, balance or ownership failure, permission limit, or any other hard stop in this Skill or the live server response.
- This is application-level transaction authorization. It does not bypass or promise to bypass any Codex platform approval that may still be required for a destructive MCP tool call.

### Goal execution loop

When a Goal covers multiple wallets or dependent actions:

1. Unless the Goal explicitly requires another safe ordering, prepare one action at a time and create its preview immediately before presenting and confirming it. Do not accumulate short-lived previews while earlier actions are waiting.
2. After `confirm_dapp_action`, use the returned `execution.executionId` to obtain the status required by the next step. Wait for a successful finalized receipt or other dependency before creating the dependent action's fresh preview.
3. Apply the Goal's retry and skip rules independently to each action. When the Goal does not resolve a failed action and no conservative in-scope choice exists, stop and ask instead of inventing a retry policy.
4. If a preview expires before confirmation, discard it and create a fresh preview. Reuse the active Goal delegation for the new exact hash; the absence of a later user message is not a blocker.
5. Record a terminal result or an authorized skip for every scoped action. Do not mark the Goal complete while required wallets or dependent actions remain unaccounted for.

## Exact token approval authorization

A user request to approve an exact token amount authorizes that approval without another confirmation question. A user request to swap also authorizes an exact approval required for that swap. This exception applies only to a live adapter's token approval action, such as `approve_token`, `approve_lvmon`, `approve_ausd`, `approve_collateral`, or `approve`. It does not authorize the swap or another state-changing action. Respect an explicit user instruction to ask before approving.

1. For a standalone approval, use the user-selected wallet, token, and exact amount. For an allowance failure during a requested swap, use the server's `TOKEN_APPROVAL_REQUIRED` error and `requiredApproval` (`adapterId`, `action`, `parameters`) for the same Project and wallet. Verify the action and parameters against live capabilities. Never infer an allowance failure or create an approval from `INTERNAL_ERROR`, a timeout, or an unknown failure.
2. Set the exact allowance required for the transaction. ERC-20 `approve` replaces allowance: when 20 tokens are already approved and a swap needs 56, approve 56, not the 36-token difference. Do not expand the amount, token, wallet, or adapter-selected spender, or grant unlimited allowance.
3. Generate and display the complete approval preview, including token, spender, amount, fee, simulation, expiry, hash, and warnings required by **Preview warning presentation**. Verify a decoded ERC-20 `approve`, an exact approval scope (`exact_amount` or `exact_open_amount`), zero native value, successful simulation, and no embedded swap. An approval's `requiresExplicitConfirmation: true` still requires a distinct `confirm_dapp_action` call with the exact hash; it does not require another user message under this exception.
4. Call `confirm_dapp_action` in the same assistant turn without asking the user to confirm, then check the Execution until a successful finalized receipt. A pending, failed, reverted, or unknown approval does not permit the dependent swap. Stop and report a terminal failure or unknown outcome; do not automatically issue another approval transaction.
5. After successful approval, request a fresh swap preview using the originally requested swap parameters. Display the complete quote and ask for swap confirmation, unless an applicable Goal delegates that confirmation. Never reuse the failed swap preview, treat approval as swap consent, or repeatedly approve if the fresh swap still reports insufficient allowance.

If recovery details are absent or conflict with live capabilities, report the mismatch instead of guessing the token, amount, or spender.

## Preview warning presentation

Display returned warnings verbatim except for the two exact nad.fun boilerplate warnings excluded in [references/nadfun.md](references/nadfun.md). Omit those sentences from user-facing prose without paraphrasing them or adding replacement generic warnings. Preserve the original structured preview, every required transaction detail, and all other returned warnings. This presentation rule does not change validation, simulation, or confirmation requirements.

## Preview and execution workflow

1. Call `list_projects` and `list_dapp_adapters` to identify an available Project, adapter, action, and its declared parameters.
2. Call `list_wallets`. When exactly one assigned wallet exists, use its `walletId`. Otherwise show only the public addresses and ask the user which address to use. Under Goal-delegated confirmation, use only the Goal-scoped addresses and select among them as required by the Goal without asking again; pass matching `walletId` values internally.
3. Call `list_dapp_markets` when the action requires a market. Call `list_dapp_positions` with the selected `walletId` when the action refers to an existing position. Outside Goal-delegated confirmation, let the user select from returned positions and collect every required action parameter; the user chooses the Token, market, or position, and the Plugin or Skill name never forces one. Under Goal-delegated confirmation that explicitly delegates decisions, choose conservative in-scope values using the rules in **Goal-delegated confirmation**. In either mode, use an adapter-declared default only when the user or active Goal did not specify that value.
4. Call `preview_dapp_action` with the selected `projectId`, `walletId`, `adapterId`, action, and declared parameters.
5. Present the complete preview in a compact, readable order: dApp/action/wallet; contract and decoded method/parameters; assets and maximum value; gas and simulation; `confirmationExpiresAt`; action hash; then warnings required by **Preview warning presentation**. Preserve the adapter's exact values and applicable warning wording, but format them for readability instead of dumping raw JSON. For a swap, also show every adapter-provided `swapQuote` value without recomputing or rounding it: exact input, quoted output, minimum output (or explicitly state that none exists), route, maximum slippage, deadline, and all available price-impact/liquidity checks. For an exit, also present the adapter-provided expected received amount, available/idle liquidity, expected transaction count, exit status, remaining exit transactions, and any distinction between available and order-locked funds.
6. When **Exact token approval authorization** or **Goal-delegated confirmation** applies, continue to step 7 in the same assistant turn without asking a confirmation question. Otherwise ask the adapter-provided `confirmationPrompt` as a direct yes/no question, end the assistant turn, and wait for a new user confirmation without demanding a fixed phrase or repetition of details.
7. After exact approval authorization, applicable Goal-delegated confirmation, or an affirmative reply in a later user message, call `confirm_dapp_action` with the exact `actionId` and `actionHash`. Never alter or recreate the preview silently. When exactly one unexpired preview is awaiting confirmation, accept brief, context-dependent affirmatives in any language, including replies such as “确认”, “确定”, “可以”, “好”, “继续”, “执行”, “没问题”, “yes”, “ok”, “go ahead”, “looks good”, or an unambiguous affirmative gesture such as “👍”. Do not ask the user to restate the dApp, action, asset, amount, or other preview details. Ask for clarification only when multiple previews could be referenced, the reply is negative or conditional, it changes any previewed field, or it is genuinely unrelated or unintelligible. Apply **UTC and confirmation-time authority** for every validity check.
8. Use `execution.executionId` returned by confirmation to poll `get_dapp_execution_status`. Report `submitted`, `outcome`, `executionStatus`, `transactionHash`, `explorerUrl`, receipt state, and any failure reason. If the Action expired or the adapter changed, create a fresh preview and validate its authorization again; swaps require a later user reply or Goal-delegated confirmation.

## Execution status lookup

When the user asks whether a recent transaction was submitted, succeeded, or failed, this is a read-only lookup and does not require the preview workflow:

1. If an `actionId` is available but no `executionId` is known, call `get_dapp_action` to retrieve the Action and associated Execution state.
2. If an `executionId` is available, call `get_dapp_execution_status` directly.
3. If the user has no ID or says “the latest/recent transaction”, call `list_dapp_executions` first. Use `projectId` only when the user named a Project or the result needs narrowing.
4. Interpret the normalized fields literally: `submitted=true` means a transaction hash was accepted; `submitted=false` means it was only queued or was rejected before broadcast; `submitted=null` means submission is still in progress or cannot be determined safely. Never describe `CREATED` as submitted.
5. `outcome=SUCCEEDED` is the only success result. `PENDING` is not success. Distinguish `FAILED` (not submitted), `REVERTED` (submitted and reverted), and `UNKNOWN` (the request may have been submitted but cannot be proved safely). Include `errorCode`/`failureReason`.
6. `retryable` is always false for an existing Execution, including `UNKNOWN`. It does not block the wallet. When the user or active Goal requests the same operation again, start a fresh `preview_dapp_action` and obtain confirmation through a later user reply or Goal-delegated confirmation; confirmation creates a new Action and Execution. Never requeue the old Execution, reuse its expired Action, or reuse its idempotency key.
7. Always return the `transactionHash` and `explorerUrl` when present. If no hash is present, say so explicitly.

## MonadVision transaction links

- Monad mainnet transaction details must use MonadVision: `https://monadvision.com/tx/<transactionHash>`.
- Whenever `transactionHash` is present, render the hash as a Markdown link to that exact URL, using the complete hash in the URL even when the visible label is shortened.
- Report the canonical MonadVision URL as `explorerUrl`. Do not display or substitute another explorer URL, even if an adapter returns a different `explorerUrl`.
- Apply this rule to every broadcast transaction, including multi-transaction exits. Do not apply it to `executionId`, `actionId`, `actionHash`, block numbers, wallet addresses, or contract addresses.
- If no `transactionHash` is present, say that no transaction hash or MonadVision link is available; never fabricate a link from another identifier.

## Mandatory swap confirmation gate

A preview is swap-related when any of these are true:

- `swapQuote` is present;
- the action exchanges assets, including action names such as `swap_*`, `buy`, `sell`, or an immediate redemption from one asset to another;
- the preview contains both an input asset and an output asset.

`requiresExplicitConfirmation` alone does not classify an action as a swap: all stored previews require a distinct confirmation API call. Exact token approvals use their authorization exception; every actual swap keeps this gate.

For every swap-related preview:

1. Treat the adapter's quote as immutable display data. Never recompute, round away, relabel, or substitute a browser price for any value.
2. Show exact input, quoted output, minimum output, route, slippage, price impact/liquidity checks, gas estimate, expiry/deadline, simulation result, and warnings required by **Preview warning presentation** before asking for confirmation.
3. If canonical `swapQuote` data is missing, inconsistent with `assets`, has zero output, or the simulation is not successful, do not confirm or execute. Report the missing/unsafe field.
4. Under Goal-delegated confirmation, present the complete swap preview and continue to `confirm_dapp_action` in the same assistant turn. Otherwise ask a direct confirmation question and stop; never call `confirm_dapp_action` in the preview turn without Goal-delegated confirmation.
5. Outside Goal-delegated confirmation, accept confirmation only from a new user message after the displayed swap. When it is the only unexpired preview awaiting confirmation, a brief conversational affirmative counts without restating the swap or quote; apply the examples and ambiguity rules in **Preview and execution workflow**. Never infer confirmation from an ordinary original request, a non-Goal standing instruction, an automation, previous consent, or permission to trade generally. An explicit active Goal that satisfies **Goal-delegated confirmation** is the sole standing-authorization exception.
6. Any changed amount, token, route, quote, minimum output, wallet, adapter version, action hash, or expired deadline invalidates the current preview and its confirmation. Obtain and display a fresh preview, then obtain confirmation through a later user reply or Goal-delegated confirmation.
7. When `minimumOutput` is `null`, explicitly say that the transaction has no enforceable minimum-output protection. Outside Goal-delegated confirmation, require the user to confirm that specific risk. Under Goal-delegated confirmation, the delegation may authorize it only after the complete warning is displayed. Do not soften or omit this warning.

The preview and server confirmation boundary remains per transaction. Exact approval authorization never doubles as swap confirmation, and confirmation of one swap never authorizes a later swap. Under Goal-delegated confirmation, the standing delegation may authorize each in-scope preview independently, but each preview still requires its own exact hash and `confirm_dapp_action` call.

## Quick-exit truthfulness

- Treat exit fields as a real-time result, never as a standing guarantee. Preserve the adapter's exact status and warning wording.
- Never describe Aave withdrawal as available when debt is present or reserve liquidity is below the requested amount.
- Kuru `withdraw_all` withdraws only available MarginAccount MON and USDC. Funds locked in open orders require a separately previewed and confirmed `cancel`, then a fresh `withdraw_all`; report two transactions when both are needed.
- Morpho `redeem` uses current idle liquidity. `emergency_force_redeem` is a different one-transaction Vault V2 multicall that uses the pinned adapter/market and must show the force-deallocated amount, penalty, simulation, and expected received amount.
- If any liquidity, ownership, balance, penalty, adapter, market, expected-received, or simulation field is absent or unsafe, do not confirm the exit. Create a fresh preview after state changes.

## Security boundaries

- Never ask for or reveal a signer key, private key, seed phrase, Privy wallet ID, Privy App Secret, or KMS credential.
- Let the MCP client complete OAuth in the browser when authentication is requested. Never ask the user to copy an access token, and never print or store OAuth tokens in project files or tool arguments.
- Never claim that the server can safely redirect a raw `/mcp` request into OAuth. The MCP client must initiate authorization so it can generate and validate its client identity, redirect URI, state, and PKCE challenge.
- Only the platform Wallet UUID and public address may be client-visible. Never expose the Privy provider wallet ID.
- Do not construct or broadcast opaque user-supplied calldata or arbitrary target addresses. The only exception is an adapter-declared calldata/router parameter, currently Uniswap `swap_exact_input`: pass it only when the user explicitly supplied it, and rely on the server to decode it and prove that the route, recipient, amount, native value, factory pools, and compatible Router02 bytecode match the normalized action and that its minimum output meets the server safety floor. A mismatch or missing decoded preview is a hard stop.
- Never bypass a disabled execution flag, owner requirement, adapter version pin, balance check, simulation failure, permission limit, or one-time confirmation.
- A configured wallet Owner does not make an App Secret harmless. Secret isolation, the exact project Owner, an authorization signer, Privy policies, scoped OAuth tokens, and transaction limits remain required.

## Connection recovery

Do not equate missing tools, temporary service unavailability, or a stale task with an expired OAuth grant. Restarting or redeploying the configured Dashboard does not by itself revoke the PostgreSQL-backed OAuth client, refresh token, or signing key.

When `trading-bot` tools are missing or unavailable in Codex:

1. Treat the MCP server configured by the active client as the only endpoint authority. Never probe a hard-coded hosted, production, staging, or localhost URL; it may be unrelated to the installed plugin configuration.
2. If the tools are absent from the current task, say that the task's MCP tool snapshot may be stale and ask the user to open a new task. Do not infer service health or OAuth state by calling an unrelated URL.
3. If a `trading-bot` tool call reports transport or service unavailability, report that exact failure against the configured MCP server. Do not claim that authorization expired and do not start OAuth.
4. Treat the grant as expired only when an MCP response or OAuth refresh explicitly reports `invalid_token`, `invalid_grant`, a revoked refresh token, or an expired refresh token. A Codex UI label such as “OAuth authenticated” only confirms that credentials are stored; it is not a live connection health check.
5. For a confirmed OAuth failure, use the active client's supported login flow and never ask the user to copy a token:
   - In Codex, run `codex mcp login trading-bot` when the user explicitly asks to reconnect.
   - In Claude Code, ask the user to open `/mcp`, select `trading-bot`, and complete authentication.
6. After authentication succeeds, ask the user to start a new Codex task or Claude Code session before retrying when the current tool snapshot remains stale.

Read [references/leverup.md](references/leverup.md) when the selected project is LeverUp, a protocol check fails, or its adapter is being maintained.

Read [references/perpl.md](references/perpl.md) when the selected project is Perpl, the wallet needs AUSD approval/account setup, a position is being closed, a protocol check fails, or its adapter is being maintained.

Read [references/nadfun.md](references/nadfun.md) when the selected project is nad.fun, a graduated V2 token needs Router approval, graduation validation fails, a protocol check fails, or its adapter is being maintained.

Read [references/uniswap.md](references/uniswap.md) when the selected project is Uniswap, any token is being approved or swapped, a custom path/pool/router/calldata is requested, a quote is rejected, or its adapter is being maintained.

Read [references/pancakeswap.md](references/pancakeswap.md) when the selected project is PancakeSwap, LVMON is being approved or swapped to MON, a quote is rejected, or its adapter is being maintained.

Read [references/aave.md](references/aave.md) when the selected project is Aave, USDC is being approved/supplied/withdrawn, debt or reserve liquidity blocks an exit, or its adapter is being maintained.

Read [references/kuru.md](references/kuru.md) when the selected project is Kuru, margin is deposited/withdrawn, a limit order is placed/canceled, an exit requires multiple transactions, or its adapter is being maintained.

Read [references/morpho.md](references/morpho.md) when the selected project is Morpho, Saturn USDC is approved/deposited/redeemed, force-deallocation is considered, or its adapter is being maintained.
