# nad.fun V2 launched-token reference for Trading Bot

- Adapter ID: `nadfun` (the Project ID comes from `list_projects`)
- Network: Monad mainnet, chain ID `143`
- User-selected resource: an exact launched-token contract address validated by the adapter and ultimately constrained by the wallet's Privy Policy
- Actions: `buy`, `approve_token`, `sell`

## Trading flow

The adapter intentionally supports only nad.fun V2 tokens that the pinned official Router reports as graduated. It does not buy or sell tokens that are still on a Bonding Curve. The Router is fixed by the audited deployment manifest and is never selected or overridden by client input.

For a buy, provide `token`, exact `monAmount`, and optional `maxSlippageBps`. For a sell, provide `token`, exact `tokenAmount`, and optional `maxSlippageBps`. Token amounts use the official 18-decimal nad.fun token format.

The adapter does not impose a fixed MON buy ceiling or Token amount ceiling. Amounts must be positive and remain subject to the assigned wallet's balance, gas, allowance, Privy Policy, live quote, minimum-output, deadline, and simulation checks.

A sell needs sufficient exact allowance for the pinned V2 Router. When allowance is insufficient, preview `approve_token` first with the same Token and intended amount, obtain a fresh explicit confirmation, wait for that transaction to finalize, and then create a new sell preview.

Every buy or sell preview is a swap preview. Display its exact input, V2 Router quoted output, minimum output, `V2_DEX` route, Router, slippage, deadline, gas and warning. Then end the turn and wait for a new explicit confirmation of that exact quote. Never confirm a buy or sell in the same turn as its preview; token approval confirmation is separate and does not authorize the sell.

## Fail-closed checks

- The official V2 Router address and runtime code hash are pinned to the audited deployment manifest.
- The Router must report the Token as graduated before the adapter requests a quote.
- Bonding Curve and otherwise ungraduated Tokens are not tradable through this adapter.
- The pinned Router must return a positive quote.
- Recipient is always the assigned wallet; approval is exact-amount only.
- Every trade has an immutable minimum output and short deadline.
- Before broadcast, the adapter repeats protocol hash, graduation, quote, deadline, balance, allowance, slippage-floor, and simulation checks.
