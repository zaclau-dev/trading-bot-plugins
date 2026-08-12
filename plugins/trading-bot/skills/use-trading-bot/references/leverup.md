# LeverUp adapter reference for Trading Bot

This reference describes adapter version `0.8.1`. Values are runtime-pinned defaults, not timeless protocol facts.

## Supported operations

- Adapter ID: `leverup` (the Project ID comes from `list_projects`)
- Chain: Monad mainnet, chain ID `143`
- Actions: `approve_collateral`, `open_position`, `close_position`
- Market: selected by the user; the adapter resolves it against the live LeverUp pair list during preview
- Direction: `long` or `short`
- Default collateral asset: native `MON`
- Default leverage: `2x`
- Default maximum slippage: `100` basis points
- Stop loss and take profit: disabled in this adapter version
- Broker ID: `1` by default and server-configurable

## Collateral assets

The adapter supports these collateral mappings:

| Collateral asset | `tokenIn` | `lvToken` | Transaction native value |
| --- | --- | --- | --- |
| `MON` | `WMON` | `LVMON` | collateral amount plus oracle fees |
| `LVMON` | `LVMON` | `LVMON` | oracle fees only |
| `LVUSD` | `LVUSD` | `LVUSD` | oracle fees only |
| `USDC` | `USDC` | `LVUSD` | oracle fees only |
| `LV` | `LV` | `LV` | oracle fees only |

The current LeverUp mainnet frontend uses the pinned WMON address as `tokenIn`
for native MON while sending the MON collateral amount as transaction value.
`LV` is enabled only when LeverUp's live direct-collateral switch, oracle, margin
token, and open-cap checks all pass.

`collateralAmount` is denominated in the selected collateral asset, not always in
MON. The adapter applies the pinned decimals and server limit for that asset.

## ERC-20 collateral approval

Before opening with `LVMON`, `LVUSD`, `USDC`, or `LV`, preview and independently
authorize a separate `approve_collateral` action for the same intended market
and collateral amount under the main Skill workflow:

```json
{
  "market": "BTC/USD",
  "collateralAsset": "USDC",
  "collateralAmount": "100"
}
```

The adapter reads the market-specific oracle update and constructs a finite
approval for `collateralAmount` plus the current `verifition_fee` raw Token units.
For example, a 2-unit fee makes a 100 USDC approval `100.000002` USDC and a
100 LVMON approval `100.000000000000000002` LVMON. Use the displayed
`approvalAmount` and `approvalAmountRaw` as the source of truth; do not recompute
them.

The spender is the pinned LeverUp Diamond, not the Open Market Facet. The Facet
executes through the Diamond, so the Diamond remains the ERC-20 `transferFrom`
spender. `MON` is native collateral and cannot be passed to
`approve_collateral`. Never use an unlimited approval, combine the approval and
open authorizations, or treat approval authorization as permission to open a
position.

After the approval succeeds, create a fresh `open_position` preview and obtain
an independent confirmation source under the main Skill. Goal delegation may
authorize both actions, but each action must use its own displayed preview and
exact hash. A pre-existing allowance is accepted only when it is at least the
displayed `requiredAllowanceAmount`, which includes the current Token fee. If
that fee changes before the open preview, create and independently authorize a
new finite approval preview.

## Open a position

Required and optional `parameters` for `preview_dapp_action`:

```json
{
  "market": "BTC/USD",
  "direction": "long",
  "collateralAsset": "USDC",
  "collateralAmount": "100",
  "leverage": 2,
  "maxSlippageBps": 100
}
```

`market`, `direction`, and `collateralAmount` are required. `collateralAsset` is
optional and defaults to `MON`; `leverage` defaults to `2`; and
`maxSlippageBps` defaults to `100`. Always use the capability declaration returned
by `list_dapp_adapters` as the live contract.

LVMON can be used directly as LeverUp collateral in adapter version `0.8.1`.
Swapping LVMON to MON through PancakeSwap is a separate optional workflow and is
not required for a LeverUp position.

## Close a position

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

## Runtime safety checks

Before generating a plan, the adapter:

- resolves the user-selected market from LeverUp's live pair list and requires `AVAILABLE` status;
- reads trading, fee, and slippage settings for the selected pair from the Diamond;
- applies per-asset decimals and collateral limits, then checks the selected wallet's balance;
- requires ERC-20 balance and allowance to the pinned LeverUp Diamond for
  `collateralAmount` plus the live `extraFee` Token units;
- pins collateral addresses, decimals, and bytecode hashes and rejects unsupported or changed token contracts;
- checks native value as collateral plus oracle fees for MON and oracle fees only for ERC-20 collateral;
- checks LeverUp's live direct-collateral switch, oracle, margin-token registration, and open cap for LV;
- checks adapter-wide leverage, market, action, and direction limits; Privy Policies provide the wallet-level authorization boundary;
- requires the exact project Owner, wallet Policy, execution signer, and signer override Policy;
- resolves `openMarketTradeV2` and `closeTrade` through Diamond Loupe and compares the facet and bytecode hash with pinned values;
- fetches the selected market price and the applicable collateral price, then fetches oracle update data for the selected pair;
- caps the total oracle fee and simulates the exact transaction from the assigned wallet.

For `approve_collateral`, the adapter requires the intended `market`, fetches its
live oracle update, and constructs only a finite ERC-20 `approve` call to the
pinned Diamond for collateral plus the current `verifition_fee` raw units, with
zero native value. It first checks the market, token identity, balance, amount,
fee ceiling, and applicable LV state.

The server repeats the contract, asset mapping, amount, allowance, native value,
facet/hash, and exact transaction simulation checks immediately before signing.

Adapter `0.8.1` pins the current LeverUp Monad trading facet used by both
`openMarketTradeV2` and `closeTrade`. The public action parameters and calldata
ABI are unchanged from `0.8.0`; this release restores compatibility after the
Diamond moved those selectors to the upgraded facet.

## Maintenance triggers

Disable live execution and update the adapter when any of these occur:

- the Diamond, WMON, LVMON, LVUSD, USDC, or LV addresses change;
- a pinned collateral token's decimals or bytecode hash changes;
- the collateral mapping, direct-collateral state, oracle, margin-token registration, or open cap changes;
- the `openMarketTradeV2` selector resolves to a different facet;
- the `closeTrade` selector resolves to a different facet;
- a pinned facet bytecode hash changes;
- LeverUp changes the pair schema, order ABI, oracle endpoints, request/response schema, fee math, collateral convention, broker convention, or chain deployment;
- the current frontend and developer documentation disagree on a transaction field;
- simulation starts failing for otherwise valid previews.

Run `check:projects` before a production release and as a recurring deployment probe.

## Operator boundary

The Plugin and Skill are distributable client instructions. Privy App credentials,
server-side authorization signer, internal wallet IDs, operator mappings, plan database,
and live-execution switch exist only in the hosted services. The Agent obtains
short-lived OAuth tokens through browser authorization; the administrator never
copies tokens or receives signing material.
