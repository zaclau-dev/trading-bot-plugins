# Perpl adapter reference for Trading Bot

Perpl is integrated on Monad mainnet (`eip155:143`) through direct wallet calls. Do not use its one-click/delegated HTTP order forwarding path.

The supported setup sequence for a new wallet is:

1. `approve_collateral` for the exact AUSD amount.
2. `create_account` with at least the current protocol minimum.
3. `open_position` with a listed market, direction, AUSD notional, leverage and slippage cap.

Existing accounts can use `deposit_collateral`. Use `list_dapp_positions` before `close_position`; the close action reads the current onchain position and constructs a full reduce-only IOC order. IOC open and close orders may fill partially when the book has insufficient liquidity before the fixed price limit.

Every Perpl preview checks the official public context against pinned addresses, reads mark price and market state onchain, pins both the EIP-1967 proxy and implementation code hashes, simulates the exact transaction, and estimates buffered Monad gas. Any address, implementation, bytecode, collateral, chain, stale price, allowance, balance, risk limit or simulation mismatch must fail closed.
