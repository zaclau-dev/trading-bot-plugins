# LeverUp adapter reference for Trading Bot

This reference describes adapter version `0.6.0`. Values are runtime-pinned defaults, not timeless protocol facts.

## Supported operation

- Adapter ID: `leverup` (the Project ID comes from `list_projects`)
- Chain: Monad mainnet, chain ID `143`
- Actions: `open_position`, `close_position`
- Market: selected by the user; the adapter resolves it against the live LeverUp pair list during preview
- Direction: `long` or `short`
- Collateral: native MON; `tokenIn` uses the current WMON address expected by LeverUp
- Default leverage: `2x`
- Default maximum slippage: `100` basis points
- Stop loss and take profit: disabled in this adapter version
- Broker ID: `1` by default and server-configurable

Required `parameters` for `preview_dapp_action`:

```json
{
  "market": "BTC/USD",
  "direction": "long",
  "collateralAmount": "100",
  "leverage": 2,
  "maxSlippageBps": 100
}
```

The market is not fixed to MON. `collateralAmount` is still denominated in native MON for every currently supported LeverUp market.

To close an entire position, pass its LeverUp `tradeHash`:

```json
{
  "tradeHash": "0x<64 hex characters>"
}
```

Call `list_dapp_positions` with the selected project, wallet, and `leverup`
adapter first; it returns only positions currently reported for that assigned wallet.

`close_position` calls the allowlisted `closeTrade(bytes32)` method with zero native
value. Preview and pre-submit simulation run from the assigned wallet, so a missing,
already closed, or foreign position is rejected. LeverUp close settlement is oracle
driven: the submitted transaction receipt proves acceptance of the close request,
not the later settlement outcome.

LVMON conversion is not a LeverUp adapter action in version `0.6.0`. When the
server advertises the PancakeSwap adapter, use its separately previewed and
confirmed `approve_lvmon` and `swap_lvmon_to_mon` actions instead. Never infer
that PancakeSwap is available without discovering it from `list_dapp_adapters`.

## Runtime safety checks

Before generating a plan, the adapter:

- resolves the user-selected market from LeverUp's live pair list and requires `AVAILABLE` status;
- reads trading, fee, and slippage settings for the selected pair from the Diamond;
- checks adapter-wide collateral, leverage, market, action, and direction limits; Privy Policies provide the wallet-level authorization boundary;
- requires the exact project Owner, wallet Policy, KMS signer and signer override Policy, then checks MON balance;
- resolves `openMarketTradeV2` and `closeTrade` through Diamond Loupe and compares the facet and bytecode hash with pinned values;
- fetches the selected market price and the MON collateral price separately, then fetches oracle update data for the selected pair;
- simulates the exact transaction from the assigned wallet.

The server repeats the facet/hash check and exact transaction simulation immediately before signing.

## Maintenance triggers

Disable live execution and update the adapter when any of these occur:

- the Diamond, collateral, or lvToken addresses change;
- the `openMarketTradeV2` selector resolves to a different facet;
- the `closeTrade` selector resolves to a different facet;
- the pinned facet bytecode hash changes;
- LeverUp changes the pair schema, order ABI, oracle endpoints, request/response schema, fee math, collateral convention, broker convention, or chain deployment;
- the current frontend and developer documentation disagree on a transaction field;
- simulation starts failing for otherwise valid previews.

Run `check:projects` before a production release and as a recurring deployment probe.

## Operator boundary

The Plugin and Skill are distributable client instructions. Privy App credentials, KMS authorization signer, internal wallet IDs, operator mappings, plan database, and live-execution switch exist only in the hosted services. The Agent obtains short-lived OAuth tokens through browser authorization; the administrator never copies tokens or receives signing material.
