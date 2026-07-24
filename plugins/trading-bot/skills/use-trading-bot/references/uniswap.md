# Uniswap Monad swap adapter reference for Trading Bot

- Adapter ID: `uniswap` (the Project ID comes from `list_projects`)
- Network: Monad mainnet, chain ID `143`
- Generic swap action: `swap_exact_input`
- Generic approval action: `approve_token`
- Backward-compatible actions: `swap_mon_to_ausd`, `swap_ausd_to_mon`, `approve_ausd`

## Generic token, path, pool, router, and calldata parameters

Always discover the live capability schema. `swap_exact_input` currently accepts:

- `tokenPath`: comma-separated token addresses, with `MON` allowed only at the first or last endpoint;
- `feeTiers`: comma-separated Uniswap v3 uint24 fee tiers, one per hop;
- `amountIn`, `tokenInDecimals`, and `tokenOutDecimals`; token decimals are checked onchain;
- optional display symbols and `maxSlippageBps`;
- optional `pools`, one address per hop, each checked against the official Uniswap v3 Factory;
- optional `router`, accepted only when its runtime bytecode matches the reviewed SwapRouter02;
- optional `calldata`, accepted only as decoded `exactInput`, one-hop `exactInputSingle`, or `multicall(exactInput,unwrapWETH9)`. Its path/fee, wallet recipients, amount, and native value must match the normalized action, and its minimum output must be at least the server-computed safety floor.

`approve_token` takes a token, exact amount, onchain-verified decimals, and optional compatible router. It never creates an unlimited approval.

The adapter does not impose a fixed AUSD or other token-amount ceiling. Inputs and approvals must be positive and remain subject to wallet balance, exact allowance, Privy Policy, live quote, minimum-output, slippage, price-impact, route, peg, gas, and simulation checks.

Do not invent token addresses, decimals, fee tiers, pools, router addresses, symbols, or calldata. Use values the user supplied or values returned by discovery. Treat `MON` and WMON as distinct except for the adapter's endpoint wrapping/unwrapping behavior.

## Verification and quote behavior

For generic routes, the server verifies canonical Factory and Quoter bytecode, compatible Router02 bytecode, token contracts and decimals, every Factory pool, current quote, size-aware price impact, allowance/balance, gas capacity, recipient, native value, slippage floor, and full simulation. It repeats transaction decoding, the quote, safety ceilings, and simulation immediately before broadcast.

The legacy MON/AUSD actions retain their additional best-route search, independent WMON/USDC pool comparison, and two-way USDC/AUSD peg check. Those legacy checks do not appear on unrelated custom token routes.

Custom tokens, pools, and routes may be illiquid, malicious, or misleading. Display the actual addresses, verified pools, Router, path, fee tiers, symbols, exact input, quote, minimum output, price impact, gas, simulation, and all warnings. Never imply that Factory registration proves a token is safe.

## Mandatory two-turn confirmation

Every swap preview contains `requiresExplicitConfirmation: true` and a canonical `swapQuote`. Display it verbatim, ask the adapter-provided confirmation question, and end the assistant turn. Never call `confirm_dapp_action` in the preview turn. Only a later explicit confirmation of that exact displayed plan permits confirmation.

An ERC-20 approval is a separate action. Approval confirmation never counts as swap confirmation, and changing any token, path, fee, pool, router, calldata, amount, quote, wallet, or adapter version requires a fresh preview and a new confirmation.
