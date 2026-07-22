# Uniswap Monad swap adapter reference for Trading Bot

- Adapter ID: `uniswap` (the Project ID comes from `list_projects`)
- Network: Monad mainnet, chain ID `143`
- Swap actions: `swap_mon_to_ausd` and `swap_ausd_to_mon`
- Approval action: `approve_ausd`

## Fixed routing and quote

The adapter never executes a direct WMON/AUSD pool. It searches only the pinned WMON/USDC/AUSD two-hop route (reversed for AUSD to MON) across reviewed Uniswap v3 fee tiers. The selected transaction uses the adapter's exact input and minimum output.

Before returning a preview and again before broadcast, the adapter checks pinned contract bytecode, a second independent WMON/USDC pool price, size-aware price impact, the two-way USDC/AUSD peg, the current best pinned route, slippage floor, wallet capacity, and full transaction simulation. Any failed check blocks the swap.

## Mandatory two-turn confirmation

The swap preview contains `requiresExplicitConfirmation: true` and a canonical `swapQuote` with exact input, quoted output, minimum output, route, slippage, plus decoded price-impact and route-deviation fields.

Display those values and all warnings, ask the user whether to execute that exact quote, and end the assistant turn. Never call `confirm_dapp_action` in the preview turn. Only a later explicit user confirmation of the displayed swap permits confirmation. AUSD approval is a separate action and never counts as swap confirmation.
