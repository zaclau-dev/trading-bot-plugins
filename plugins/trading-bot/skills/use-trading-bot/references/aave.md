# Aave V3 supply-only adapter reference

Adapter version `0.1.0` is pinned to Aave V3 Monad mainnet USDC.

- Adapter ID: `aave`
- Actions: `approve`, `supply`, `withdraw`
- Asset: official Monad USDC, 6 decimals
- Borrow, collateral-toggle, repay, liquidation, and arbitrary reserve actions are not exposed.

Use an exact `approve` before `supply` when allowance is insufficient. Supply-only mode refuses a wallet that already has Aave debt. A `withdraw` preview reads the wallet aUSDC balance, total Aave debt, and current USDC reserve liquidity. It is executable only when debt is zero, the requested amount is covered by aUSDC, reserve liquidity is sufficient, and the exact transaction simulates.

Display `expectedReceivedUsdc`, `availableLiquidityUsdc`, `expectedTransactions`, and `immediateExitStatus`. These are real-time results. Never say Aave guarantees future liquidity.

The adapter pins Pool/aUSDC proxies, their EIP-1967 implementations, Pool implementation bytecode, USDC bytecode, chain ID, exact asset, beneficiary/receiver, referral code zero, and exact approval spender/amount. It rechecks and simulates immediately before submission.
