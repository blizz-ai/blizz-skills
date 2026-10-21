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

Load the key into the shell that starts Codex — for this session only, not
into `~/.zshrc` or any file:

```bash
read -rs BLIZZ_API_KEY && export BLIZZ_API_KEY   # paste, Enter; nothing is echoed
# or from the macOS keychain:
# export BLIZZ_API_KEY=$(security find-generic-password -s blizz-api -w)
```

Never paste the key into a chat with Codex or any AI. If it ever shows up in
one, revoke it on blizz.id under **API**.

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

## Updating

Read what changed before you take it — the skills tell Codex what to do with
your money:

```bash
cd ~/.codex/blizz-skills && git fetch && git log -p HEAD..origin/main -- skills
git merge --ff-only origin/main
```

## Uninstalling

```bash
rm ~/.agents/skills/blizz
rm -rf ~/.codex/blizz-skills
```

Then remove the `[mcp_servers.blizz]` block from `~/.codex/config.toml`.
