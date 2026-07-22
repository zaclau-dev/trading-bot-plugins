# PancakeSwap Monad adapter reference for Trading Bot

This reference describes adapter version `0.1.0`. Values are runtime-pinned
defaults, not timeless protocol facts.

- Adapter ID: `pancakeswap` (the Project ID comes from `list_projects`)
- Network: Monad mainnet, chain ID `143`
- Market: the pinned LVMON/WMON PancakeSwap v3 0.25% pool
- Approval action: `approve_lvmon`
- Swap action: `swap_lvmon_to_mon`

## Approval and swap flow

For an exact LVMON amount:

1. When allowance is insufficient, preview `approve_lvmon` with
   `lvmonAmount`, display the exact Router allowance, obtain a separate explicit
   confirmation, and wait for that approval to finalize.
2. Create a new `swap_lvmon_to_mon` preview with the same `lvmonAmount` and
   optional `maxSlippageBps`.
3. Display the adapter-provided exact input, quoted MON output, enforceable
   minimum MON output, fixed route, pool, price impact, slippage, deadline,
   network fee, simulation result, and all warnings.
4. Ask the adapter-provided confirmation question and end the turn. Only a new
   explicit confirmation of that exact quote permits `confirm_dapp_action`.

The fixed route is LVMON to WMON through the pinned PancakeSwap v3 pool, followed
by WMON unwrapping to native MON in the same Router multicall. Approval
confirmation never authorizes the later swap.

## Fail-closed checks

The adapter pins the Factory, Quoter, SwapRouter, WMON, LVMON, and LVMON/WMON
pool code hashes. Before preview and again before submission it verifies:

- Monad chain ID and the pinned pool's Factory, Token pair, fee tier, and
  non-zero liquidity;
- exact LVMON balance and Router allowance;
- Quoter output, non-zero minimum output, server slippage limit, price-impact
  limit, and short deadline;
- the exact `multicall(exactInputSingle, unwrapWETH9)` target, Token pair,
  recipient, amount, and value;
- full transaction simulation and sufficient MON for network fees.

Never accept a user-selected path, pool, Token pair, Router, target, or calldata.
