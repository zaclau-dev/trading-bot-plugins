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

## Dashboard BNCB market registration

Dashboard nad.fun V2 discovery also checks fixed BNCB quote identities: Monad mainnet (143) `0x094aDB9ab02Ed230697297C219073DC8C69Df0DF` and Monad Testnet (10143) `0x4C0920638766D229B99aE7B75E7187cE798F567C`. The testnet contract reports BNC4; nad.fun and Dashboard use the BNCB venue label. Addresses and runtime code hashes are pinned in service configuration. Existing WMON/LVMON discovery remains supported.

The mainnet BNCB token is token0 of user-selected PancakeSwap reference pool `0x1d5F026CC3Ba68da7251976d65c61a7351667A3A`; that reference pool is not a nad.fun market or a token address. Market registration still requires a graduated base token and an eligible pool from the pinned nad.fun Factory.

Order Engine execution capability is derived from the registered market network and venue. New non-equivalent quotes support direct-quote LIMIT_PRICE orders and ACCUMULATION, TWAP_SELL, SELL_TRIGGER_BUY and BUY_TRIGGER_SELL strategies on nad.fun V2 on Monad mainnet (143) and Testnet (10143). Quote, wallet and receipt reads select the RPC and pinned Router for that market chain. New-quote Grid and IMMEDIATE remain unsupported; other networks or venues report QUOTE_EXECUTION_UNSUPPORTED.

Testnet BNCB-quoted markets additionally support MON buys through the pinned Router's path-bearing `buyWithNative` overload (selector `0x704edbd9`). For orders without an explicit payment choice, whole-order BNCB funding is tried first; only proven BNCB insufficiency permits evaluating MON. A failed balance read does not permit fallback. The Router performs conversion in one transaction. MON principal and Gas share the wallet's available native balance; the execution uses fresh routed quotes, immutable minimum output, a deadline, and simulation. Existing strategy and automatic-payment limits and budgets remain BNCB-denominated; manual payment selection is described below. Receipts record actual MON cash spent separately from BNCB-valued acquisition cost and intermediate BNCB input. **SELL receives BNCB, not MON**; `sellToNative` is not enabled for these markets. Mainnet BNCB native buys are not enabled by this change.

Capability is not proof that a trade will succeed: market status, live chain/Router/pool identity, decimals, whole-order wallet balance, allowance, gas, fresh quote, and settlement checks still apply. Budget accounting uses the market quote identity and decimals independently of execution capability. This Dashboard addition does not add a testnet MCP action or change the MCP buy/sell contract above.

## Dashboard manual buy payment selection

For supported BNCB markets, the Dashboard manual limit buy accepts a fixed target-token quantity and a limit price in the selected MON or BNCB payment asset. The Order API persists optional `paymentAsset` (`MON` or `QUOTE`) and `paymentAmountAtomic`. Explicit choices never fall back to another payment asset; gas remains separately reserved in MON. The minimum base output rounds up to enforce the selected-asset limit. Split orders preserve the exact total requested target-token quantity; each payment cap is derived from that quantity and the selected-asset limit. Orders without these fields, including existing strategy Orders, retain their original quote-based behavior. These fields belong to the Dashboard Order Engine API, not the MCP `nadfun` action parameters above.
