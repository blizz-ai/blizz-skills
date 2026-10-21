# Installing Blizz for Codex

Two parts, use either or both:

- **MCP server** — Codex talks to Blizz directly (search and assess wallets, run copy orders).
- **Skills** — workflow instructions Codex loads on demand.

## Prerequisites

- A Blizz account at [blizz.id](https://blizz.id).
- An API key: on blizz.id press **API** in the header → **New key**. Pick the access you want to give the agent:
  - **Wallet data · read** — search and assess wallets;
  - **Account · read** — your wallets, balances and positions;
  - **Copy orders · read / write**, **Snipe orders · read / write** — read or manage orders.

  The secret (`sk_…`) is shown once.

## 1. MCP server

Put the key into your environment (for example `~/.zshrc`):

```bash
export BLIZZ_API_KEY=sk_...
```

Add the server to `~/.codex/config.toml`:

```toml
[mcp_servers.blizz]
url = "https://blizz.id/mcp"
bearer_token_env_var = "BLIZZ_API_KEY"
```

Restart Codex and ask: "List my Blizz copy orders".

## 2. Skills

```bash
git clone https://github.com/blizz-ai/blizz-skills ~/.codex/blizz-skills
mkdir -p ~/.agents/skills
ln -s ~/.codex/blizz-skills/skills ~/.agents/skills/blizz
```

**Windows (PowerShell):**

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.agents\skills"
cmd /c mklink /J "$env:USERPROFILE\.agents\skills\blizz" "$env:USERPROFILE\.codex\blizz-skills\skills"
```

Restart Codex. You should see three skills: `blizz`, `blizz-wallet-scout`, `blizz-copy-pilot`.

## Dev environment

The Blizz dev site is `https://jizzus.xyz`. Use `url = "https://jizzus.xyz/mcp"` and a key created there, or set `BLIZZ_API_BASE=https://jizzus.xyz` for the REST fallback.

## Updating

```bash
cd ~/.codex/blizz-skills && git pull
```

## Uninstalling

```bash
rm ~/.agents/skills/blizz
rm -rf ~/.codex/blizz-skills
```

Then remove the `[mcp_servers.blizz]` block from `~/.codex/config.toml`.
