# Morpho Saturn USDC Vault V2 adapter reference

Adapter version `0.1.1` is pinned to the listed, warning-free Saturn USDC Morpho Vault V2 on Monad mainnet.

- Adapter ID: `morpho`
- Actions: `approve`, `deposit`, `redeem`, `emergency_force_redeem`
- Vault share: `satUSDC`, 18 decimals
- Underlying: official USDC, 6 decimals

Use exact `approve` before `deposit` when needed. `deposit` displays current expected shares but does not promise a future share price or exit.

`redeem` is the normal path and is allowed only when current vault idle USDC covers the previewed share redemption. Display expected USDC, idle liquidity, one expected transaction, and the immediate exit status.

`emergency_force_redeem` is used only when idle liquidity is insufficient. It builds one atomic Vault V2 `multicall`: a `forceDeallocate` from the single pinned Morpho Market V1 adapter/market, then an exact-share `redeem` to the same wallet. Display expected USDC, idle liquidity, force-deallocated USDC, the current penalty, one expected transaction, simulation, and exit status. Never call it a guaranteed cash exit: underlying market liquidity remains live and the transaction must simulate again before submission.

The adapter pins Vault, USDC, adapter bytecode, asset/decimals, adapter count/address, parent Vault, and the emergency market tuple. Any drift in those deployment facts fails closed.

Version `0.1.1` scopes the live allocation and force-deallocation penalty checks to `emergency_force_redeem`. A nonzero penalty or empty emergency position does not block `approve`, `deposit`, or a normal `redeem` backed by idle liquidity. Emergency previews and submission still require zero penalty and enough allocation in the pinned market; a changed penalty rejects the emergency exit with its current percentage. Treat this rejection as an unsupported emergency path, not as permission to accept a fee or change the market.
