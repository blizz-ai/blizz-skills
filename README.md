# Blizz for AI agents

Connect Claude, ChatGPT or your own agent to **[Blizz](https://blizz.id)**, the
Solana copy-trading service. Your agent can:

- **find wallets worth copying** — search Solana wallets by profit, win rate,
  activity and behaviour;
- **assess a wallet** — a deterministic verdict (copy at a small size / copy
  the buys with your own stops / watch / do not copy), four gates with their
  numbers, a tier and a ready copy-order draft;
- **check a deployer** before sniping its next coin;
- **run your copy trading** — create, edit, pause, resume and delete copy and
  snipe orders, one or all at once;
- **read the results** — PnL, win rate and capture per order, the trade feed,
  open positions, your wallets and balances.

It cannot move or withdraw funds, see keys, create wallets or trade by hand —
those functions do not exist in the API.

Guide on the site: **https://blizz.id/ai**.

## Connect

### Claude (claude.ai, Desktop, mobile)

Settings → **Connectors** → **Add custom connector** → name `Blizz`, URL:

```
https://blizz.id/mcp
```

→ **Connect** → sign in to Blizz → choose what Claude may do (and the most it
may spend per order) → **Allow**. Works on every plan (Free: one custom
connector). On Team and Enterprise an owner adds it for the organisation.
Mobile apps use connectors added on the web or Desktop.

### ChatGPT

Settings → **Plugins** (Apps & Connectors) → **Add custom MCP server** → URL
`https://blizz.id/mcp` → sign in with OAuth. Reading works on Pro; creating
and changing orders works only on **Business, Enterprise and Edu** workspaces
(web).

### Claude Code

```bash
claude mcp add --transport http blizz https://blizz.id/mcp
```

then `/mcp` → **Authenticate**. Or install the plugin — the MCP server and
the three skills in one go:

```
/plugin marketplace add blizz-ai/blizz-skills
/plugin install blizz@blizz
```

### Cursor

`~/.cursor/mcp.json`:

```json
{ "mcpServers": { "blizz": { "url": "https://blizz.id/mcp" } } }
```

Cursor signs in with OAuth. With an API key instead:

```json
{ "mcpServers": { "blizz": { "url": "https://blizz.id/mcp",
  "headers": { "Authorization": "Bearer ${env:BLIZZ_API_KEY}" } } } }
```

### Codex CLI

`~/.codex/config.toml`:

```toml
[mcp_servers.blizz]
url = "https://blizz.id/mcp"
bearer_token_env_var = "BLIZZ_API_KEY"
```

Skills for Codex: [`.codex/INSTALL.md`](.codex/INSTALL.md).

### VS Code (Copilot)

`.vscode/mcp.json`:

```json
{ "servers": { "blizz": { "type": "http", "url": "https://blizz.id/mcp" } } }
```

### Gemini CLI

```bash
gemini mcp add --transport http blizz https://blizz.id/mcp --header "Authorization: Bearer $BLIZZ_API_KEY"
```

### OpenClaw and any agent with skills

```bash
npx skills add blizz-ai/blizz-skills
export BLIZZ_API_KEY=sk_...          # an API key from blizz.id
export BLIZZ_API_BASE=https://blizz.id
```

Without MCP the skills call the REST API with the key
([`skills/blizz/references/rest-api.md`](skills/blizz/references/rest-api.md)).

### API keys

On [blizz.id](https://blizz.id) press **API** in the header → **New key** and
pick the access:

| access | lets the agent |
|---|---|
| Wallet data · read | search and assess wallets, check deployers |
| Account · read | see your wallets, balances and positions |
| Copy orders · read / write | read copy orders and their results / create, edit, pause, delete them |
| Snipe orders · read / write | the same for snipe orders |

The secret (`sk_…`) is shown once. Treat it as a password.

## Tools

| Tool | What it does |
|---|---|
| `search_wallets` | find wallets by profit, win rate, coins, snipe share, profit concentration, activity, hold time |
| `assess_wallet` | verdict, four gates (record is real, still making money, you can follow it, cuts losses), tier, metrics, copy-order draft |
| `get_wallet_coins` | every coin a wallet traded: SOL in/out, still held, PnL |
| `get_wallet_coin_trades` | each trade a wallet made in one coin |
| `get_wallet_pnl` | realized PnL by day for a month |
| `get_deployer` | a deployer's last 50 coins: migrations, ATH market caps |
| `list_my_wallets` | your Blizz wallets and balances |
| `list_positions` | your open positions with value and PnL |
| `list_orders`, `get_order` | your copy or snipe orders |
| `get_order_stats` | results per order for a period: copied, missed, capture, PnL, win rate, the target's own PnL |
| `list_order_trades` | the trade feed of one order: landed, failed, missed, skipped (with the reason) |
| `create_copy_order`, `create_snipe_order` | create an order |
| `update_order` | change any part of an order |
| `set_orders_state` | turn orders on or off — some or all |
| `delete_orders` | delete orders |

The tools you get depend on the access you granted. Read tools run freely;
your assistant asks before every write.

## Skills

| Skill | For |
|---|---|
| [`blizz`](skills/blizz/SKILL.md) | the core: connecting, tools, conventions, safety rules |
| [`blizz-wallet-scout`](skills/blizz-wallet-scout/SKILL.md) | search → assess → honest shortlist → confirmed copy order |
| [`blizz-copy-pilot`](skills/blizz-copy-pilot/SKILL.md) | review results, find underperformers, propose changes, daily check, kill switch |

## Try asking

```
Find 5 wallets worth copying: steady swing traders, active today, no snipers.
Is 4BdKaxN8… worth copying? Explain the verdict.
Copy the best one with 0.1 SOL per trade from my Main wallet, stop-loss 40%.
How did my copy orders do this week? Which ones lose money?
Pause every copy order whose target lost money over the last 7 days.
Stop all copying now.
Check this deployer before I snipe its next launch: 7xK…
```

## Demos

Single-file pages on the public API — open them, paste an API key, or look
around in demo mode with sample data:

- **Wallet Analyst** — search with presets, the assessment with its gates, a
  PnL calendar, coins, and "copy this wallet" with a confirmation:
  [blizz.id/ai/demos/wallet-analyst.html](https://blizz.id/ai/demos/wallet-analyst.html)
  · [source](demos/wallet-analyst/index.html)
- **Copy Cockpit** — your copy orders with results for 1D / 7D / 30D,
  on/off switches, size edits, pause all, positions and the trade feed:
  [blizz.id/ai/demos/copy-cockpit.html](https://blizz.id/ai/demos/copy-cockpit.html)
  · [source](demos/copy-cockpit/index.html)

Both run in your browser only: no backend, no build, no dependencies. The key
stays in the browser tab (or on the device if you tick "remember").

## Dev environment

The Blizz dev environment is `https://jizzus.xyz`: use
`https://jizzus.xyz/mcp`, set `BLIZZ_MCP_URL=https://jizzus.xyz/mcp` for the
Claude Code plugin and `BLIZZ_API_BASE=https://jizzus.xyz` for the REST
fallback, with a key created on the dev site.

## Security and disclaimer

- **Real money.** Orders your agent creates or turns on trade real SOL from
  your wallets. Solana transactions are irreversible. Your assistant must ask
  you before every write; read what it proposes.
- **Limits you set.** When you connect Claude or ChatGPT you choose the
  access and the most it may spend per order; Blizz enforces the cap on its
  side. You can see and disconnect connected apps at
  [blizz.id/ai](https://blizz.id/ai) and revoke API keys under **API**.
- **No funds or keys.** The agent cannot withdraw, transfer, see keys or
  create wallets — the API has no such functions.
- **Prompt injection.** Coin names, wallet labels and other on-chain text can
  contain instructions. The skills tell the agent to treat them as data, but
  stay alert: an unexpected order proposal is a red flag.
- **Past results are not future returns.** Assessments describe what a wallet
  did, not what copying it will make.

Use at your own risk. Blizz is not liable for losses caused by model
mistakes, prompt injection, leaked keys or confirmed orders.

## License

MIT
