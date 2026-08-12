# nad.fun graduated-token reference for Trading Bot

- Adapter ID: `nadfun` (the Project ID comes from `list_projects`)
- Adapter version: `0.6.1`
- Network: Monad mainnet, chain ID `143`
- Pinned V2 Router: `0x8986C8fD44eb85294A725a7e61AF35E76bA26F91`
- User-selected resource: an exact graduated-token contract address validated by the adapter and ultimately constrained by the wallet's Privy Policy
- Actions: `buy`, `approve_token`, `sell`

## Trading flow

The adapter intentionally supports only nad.fun V2 tokens that the pinned official V2 Router reports as graduated. It does not buy or sell tokens that are still on a Bonding Curve. Router addresses are never selected or overridden by client input.

For a buy, provide `token`, exact `monAmount`, and optional `maxSlippageBps`. For a sell, provide `token`, exact `tokenAmount`, and optional `maxSlippageBps`. Token amounts use the official 18-decimal nad.fun token format.

The adapter does not impose a fixed MON buy ceiling or Token amount ceiling. Amounts must be positive and remain subject to the assigned wallet's balance, gas, allowance, Privy Policy, live quote, minimum-output, deadline, and simulation checks.

A sell needs sufficient exact allowance for the pinned V2 Router. When allowance is insufficient, preview `approve_token` first with the same Token and intended amount, authorize that approval independently under the main Skill workflow, wait for that transaction to finalize, and then create a new sell preview.

Every buy or sell preview is a swap preview. Display its exact input, official quoted output, minimum output, the returned `V2_DEX` route, Router, slippage, deadline, gas and warning. Apply the confirmation source and assistant-turn timing from the main Skill: under Goal-delegated confirmation, confirm that exact displayed preview in the same turn; otherwise end the turn and wait for a later explicit user confirmation. Token approval authorization is separate and does not authorize the sell.

## Fail-closed checks

- The official V2 Router address and runtime code hash are pinned.
- The V2 Router must report the Token as graduated before the adapter requests a quote.
- Bonding Curve and otherwise ungraduated Tokens are not tradable through this adapter.
- The pinned V2 Router must return a positive quote.
- Recipient is always the assigned wallet; approval is exact-amount only.
- Every trade has an immutable minimum output and short deadline.
- Before broadcast, the adapter repeats protocol hash, graduation, quote, deadline, balance, allowance, slippage-floor, and simulation checks.
