# Blizz order object

The same object for copy and snipe orders; filters differ. Source of truth:
`https://blizz.id/api-docs` → **Order object**.

```json
{
  "id": 2415,
  "target": "<target wallet>",
  "wallet": "<your Blizz wallet>",
  "label": "scout pick",
  "state": "active",
  "buy": {
    "amount_sol": "0.5",
    "slippage_pct": 20,
    "slippage_sol": null,
    "once_per_token": false,
    "target_amount_sol": { "min": "1", "max": "5" },
    "market_cap_usd": { "min": null, "max": 50000 },
    "tip_sol": "0.001",
    "priority_fee_sol": "0.0005"
  },
  "sell": {
    "slippage_pct": 10,
    "tip_sol": "0.001",
    "priority_fee_sol": "0.0005",
    "auto": [
      { "type": "tp", "trigger_pct": 100, "size_pct": 50 },
      { "type": "sl", "trigger_pct": 30, "size_pct": 100 }
    ],
    "timed": [ { "after_sec": 600, "size_pct": 100 } ]
  },
  "filters": { "dexes": ["pump_fun", "pump_swap"], "quote": "all", "copy_sells": true, "reverse": false,
               "skip_add_ons": false, "skip_creator_launch": false, "anti_pvp": null },
  "created_at": "2026-10-08T10:00:00Z",
  "updated_at": "2026-10-08T10:05:00Z"
}
```

`id`, `created_at`, `updated_at` are read-only. `target` and `wallet` are set
on create only — to change them, delete the order and create a new one.

## Rules

- One copy order and one snipe order per target at most; one wallet holds
  one order per target. A snipe and a copy order on the same target must
  trade from different wallets.
- `wallet` must be one of the user's Blizz wallets (`list_my_wallets`); the
  target cannot be one of them.
- SOL amounts are decimal **strings**; percents and seconds are whole numbers.
- Unknown fields are rejected.
- Changes reach the trading nodes immediately; an order does not need to be
  paused to be edited.
- PATCH: nested blocks merge field by field; arrays (`sell.auto`,
  `sell.timed`, `filters.dexes`) are replaced whole; `null` or `[]` turns
  `sell.auto` / `sell.timed` off.
- Pause and resume through `state`. A paused order makes no new trades but
  keeps TP/SL and timers on its open positions.
- Deleting stops the order; open positions stay in the wallet.
- 1000 orders per account across all types.

## Fields

| field | values | default | meaning |
|---|---|---|---|
| `target` | address | required | wallet to copy, or deployer to snipe |
| `wallet` | address | required | the user's Blizz wallet that trades |
| `label` | string ≤ 32 or `null` | none | a note |
| `state` | `active`, `paused` | `active` | paused orders do not trade |
| `buy.amount_sol` | SOL string > 0 | required | most we buy per target buy (copy buys the smaller of this and what the target spent) |
| `buy.slippage_pct` | 0–100 | 20 | price tolerance on buy; 100 turns protection off |
| `buy.slippage_sol` | SOL string or `null` | `null` | SOL other buyers may add to the pool between the target's trade and ours; `"0"` buys only if nobody got in between |
| `buy.once_per_token` | bool | false | buy each coin only once |
| `buy.pause_after_first` | bool | false | **snipe only**: pause the order after its first snipe |
| `buy.fast_create_pause_min` | whole minutes 0–10080 | 0 (off) | **snipe only**: the deployer launches again sooner than this after our last snipe → skip and pause |
| `buy.target_amount_sol` | `{min, max}` SOL strings or `null` | none | copy: the target's buy; snipe: the creator's own buy. Outside → skip |
| `buy.market_cap_usd` | `{min, max}` whole dollars or `null` | none | coin market cap range |
| `buy.tip_sol` | SOL string | service minimum | tip on buy |
| `buy.priority_fee_sol` | SOL string | 0.0005 | priority fee on buy (min 0.00001) |
| `sell.slippage_pct` | 0–100 | 10 | price tolerance on sell |
| `sell.tip_sol` | SOL string | service minimum | tip on sell |
| `sell.priority_fee_sol` | SOL string | 0.0005 | priority fee on sell |
| `sell.auto` | up to 8 rows or `null` | off | take-profit / stop-loss |
| `sell.timed` | up to 5 points or `null` | off | sell by time |

`sell.auto` row: `type` `tp` | `sl`; `trigger_pct` — `tp` 1–9999 (up from
entry), `sl` 1–99 (down from entry); `size_pct` 1–100 of the remaining
position.

`sell.timed` point: `after_sec` 1–86400 (first from the buy, each next from
the previous sell); `size_pct` 1–100 of the remaining position.

## Copy filters

| field | values | default | meaning |
|---|---|---|---|
| `filters.dexes` | `pump_fun`, `pump_swap`, `mayhem`, `raydium_cpmm`, `raydium_amm_v4`, `raydium_clmm`, `raydium_launchlab`, `stonk_fun`, `orca_v1`, `orca_v2`, `orca_whirlpool`, `meteora_dlmm`, `meteora_damm_v1`, `meteora_damm_v2` | `["pump_fun","pump_swap"]` | venues to copy |
| `filters.quote` | `all`, `sol`, `non_sol` | `all` | quote currency of the coin |
| `filters.copy_sells` | bool | true | also copy the target's sells |
| `filters.reverse` | bool | false | buy when the target sells, never copy its buys; not with `copy_sells: true` or `anti_pvp` |
| `filters.skip_add_ons` | bool | false | skip buys while we still hold the coin; not with `buy.once_per_token` |
| `filters.skip_creator_launch` | bool | false | skip coins the target launched itself |
| `filters.anti_pvp` | `{hold_sec, ban_sec}` or `null` | off | the target sold within `hold_sec` (1–3600) of buying → stop copying its buys for `ban_sec` (1–86400) |

## Snipe filters

| field | values | default | meaning |
|---|---|---|---|
| `filters.dexes` | `pump_fun`, `mayhem`, `raydium_launchlab`, `stonk_fun`, `bonk_fun` | `["pump_fun"]` | launch venues |
| `filters.quote` | `all`, `sol`, `non_sol` | `all` | quote currency of the coin |
| `filters.max_token_tax_pct` | 0–100, up to 2 decimals | 5 | skip coins with a higher transfer tax |

## Sensible starting points

These are starting points to discuss with the user, not advice.

- **Size:** start small (the user's choice, within the connection cap); raise
  only after a week of results in `get_order_stats`.
- **A wallet that does not cut losses** (`assess_wallet` verdict
  `copy_buys_set_stops`): `filters.copy_sells: false` plus `sell.auto` with a
  stop-loss (e.g. `{"type":"sl","trigger_pct":40,"size_pct":100}`) and a
  take-profit ladder.
- **A target that flips fast:** `filters.anti_pvp` `{"hold_sec": 30, "ban_sec": 3600}`.
- **A target that also launches coins:** `filters.skip_creator_launch: true`.
- **Crowded entries:** `buy.slippage_sol` limits how much others may buy in
  between; `buy.market_cap_usd.max` keeps out late entries.
