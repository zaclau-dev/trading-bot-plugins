# Kuru MON/USDC adapter reference

Adapter version `0.1.1` is pinned to Kuru's official Monad mainnet MON/USDC orderbook and MarginAccount.

- Adapter ID: `kuru`
- Actions: `approve`, `deposit`, `limit_order`, `cancel`, `withdraw_all`
- Margin assets: native MON and official USDC
- Market: `MON-USDC` only

`approve` grants an exact USDC amount only to MarginAccount. `deposit` accepts an exact MON or USDC amount. `limit_order` accepts `side`, `price`, and `size`; the adapter always encodes post-only `true`, enforces live tick/min/max precision, server notional/size ceilings, available margin, and simulation.

`cancel` accepts comma-separated order IDs, verifies every order is active and owned by the assigned wallet, and returns funds to MarginAccount. It does not return funds to the wallet. A complete open-order exit is therefore two separately previewed and confirmed transactions: `cancel`, then a fresh `withdraw_all`.

`withdraw_all` sends all currently available MarginAccount MON and USDC back to the wallet in one transaction. It cannot withdraw order-locked funds. Display `expectedReceivedMon`, `expectedReceivedUsdc`, `expectedTransactions`, `remainingExitTransactions`, and `immediateExitStatus` literally.

The adapter pins proxy and implementation bytecode, EIP-1967 implementations, verified-market status, USDC bytecode, and every MON/USDC market parameter. It decodes ownership and calldata, then simulates again before submission.

Version `0.1.1` refreshes the implementation addresses and runtime code hashes observed at Monad mainnet block `112416815`, preserving the existing actions and limits. An `implementation changed` rejection requires a new review and adapter update; create a fresh preview after the updated server is running. The adapter continues to reject unknown implementations during preview and before submission.
