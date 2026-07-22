# nad.fun V3 reference for Trading Bot

- Adapter ID: `nadfun` (the Project ID comes from `list_projects`)
- Network: Monad mainnet, chain ID `143`
- User-selected resource: an exact token contract address validated by the adapter and ultimately constrained by the wallet's Privy Policy
- Actions: `buy`, `approve_token`, `sell`

## Trading flow

The adapter follows the official V3 integration: it asks the pinned Lens for both the quote and current Router. Lens may select the Bonding Curve Router before graduation or the DEX Router after graduation. Never choose or override the Router in client input.

For a buy, provide `token`, exact `monAmount`, and optional `maxSlippageBps`. For a sell, provide `token`, exact `tokenAmount`, and optional `maxSlippageBps`. Token amounts use the official 18-decimal nad.fun token format.

A sell needs sufficient exact allowance for whichever Router Lens currently selects. When allowance is insufficient, preview `approve_token` first with the same Token and intended amount, obtain a fresh explicit confirmation, wait for that transaction to finalize, and then create a new sell preview. Graduation can change the selected Router, so never reuse an approval or preview blindly across a route change.

Every buy or sell preview is a swap preview. Display its exact input, Lens quoted output, minimum output, selected Bonding Curve/DEX route, Router, slippage, deadline, gas and warning. Then end the turn and wait for a new explicit confirmation of that exact quote. Never confirm a buy or sell in the same turn as its preview; token approval confirmation is separate and does not authorize the sell.

## Fail-closed checks

- Lens, Bonding Curve Router, and DEX Router addresses and runtime code hashes are pinned.
- Lens must return one of the two pinned Routers and a positive quote.
- A Token locked between its curve and DEX graduation is not tradable.
- Recipient is always the assigned wallet; approval is exact-amount only.
- Every trade has an immutable minimum output and short deadline.
- Before broadcast, the adapter repeats protocol hash, route, quote, deadline, balance, allowance, slippage-floor, and simulation checks.
