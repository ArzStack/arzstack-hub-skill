---
name: arzstack-hub
description: >-
  Discover and install items — skills, MCP servers, knowledge, and automations —
  from the user's ArzStack Hub (their organization's governed, curated catalog)
  using the `npx @arzstack/hub` CLI, on the user's behalf. Use this skill
  whenever the user wants to install, find, browse, or set up a skill / MCP /
  tool "from the hub", "from the catalog", "from ArzStack", or asks what's
  available in their organization's catalog — even if they don't name the CLI.
  Also use it when an install fails because the CLI isn't logged in (needs a
  hub URL + PAT). Do NOT use it for unrelated `npm install` of public packages,
  nor for authoring brand-new skills from scratch.
---

# ArzStack Hub — install from the catalog

ArzStack Hub is a multi-tenant catalog where an organization curates and governs
the **skills, MCP servers, knowledge, and automations** its people are allowed to
use. The `@arzstack/hub` CLI lets you (the agent) browse that catalog and install
items for the user without them having to copy long commands from the web UI.

Your job: when the user wants something from their hub, drive the CLI for them —
authenticate if needed, find the right item, install it to the right place, and
confirm what happened.

## The one tool you need

Everything runs through `npx @arzstack/hub <command>` (no global install needed):

| command | what it does |
|---|---|
| `npx @arzstack/hub login` | saves the hub URL + a personal token (PAT) to `~/.config/arzstack/config.json` |
| `npx @arzstack/hub install` *(no argument)* or `browse` | **interactive**: lists item types with counts → paginated items → pick one → choose project/global → installs |
| `npx @arzstack/hub install <id> [--global\|--project] [--dir <path>] [--yes]` | installs a specific item directly (you already know its id) |
| `npx @arzstack/hub install <token>` | installs from an ephemeral token copied from the web UI (no login needed) |

Flags worth knowing: `--global` (default) installs to `~/.agents/skills` (shared
across Claude Code, Codex, and Gemini); `--project` installs to `./.agents/skills`
(only this repo). `--yes` skips the confirmation prompt; `--dir <path>` overrides
the destination.

## If the hub is also connected as an MCP server

The same package doubles as an MCP server (`npx @arzstack/hub mcp`). If the user's
agent host is configured with it, **you already have these tools natively** — no
shelling out to discover the catalog:

| MCP tool | what it returns |
|---|---|
| `catalog_counts` | how many items exist per type (skill, mcp_server, knowledge, automation) |
| `search_catalog` | a page of items (filter by `type`, paginate with `cursor`, optional text `query`) |
| `get_item` | install details for one id — type, source repo, and a `how_to_install` hint |

When these tools are available, **prefer them for discovery** — it's cleaner and
more reliable than parsing interactive `browse` output. Call `catalog_counts` /
`search_catalog` to find the item and `get_item` to read its id and install hint,
**then run the real install with the CLI** (`npx @arzstack/hub install <id>
--global`). The MCP is read-only: it finds and explains; the CLI installs.

The MCP uses the **same auth** as the CLI (the PAT), so once the user is set up,
nothing extra is needed. They enable it once in their agent's config — the portal
shows a ready-to-paste snippet under **Perfil → Tokens de CLI** (equivalent to
`claude mcp add arzstack-hub --env ARZSTACK_HUB=… --env ARZSTACK_TOKEN=… -- npx -y
@arzstack/hub mcp`). If you **don't** have these tools, ignore this section and
just drive the CLI as described below — nothing changes.

## Step 1 — Make sure the CLI is authenticated

Item ids and the catalog are scoped to the user's organization, so the CLI needs
a logged-in session (the ephemeral-token path is the exception).

Check whether `~/.config/arzstack/config.json` exists and contains a `hub` and a
`token`. If it does, you're ready — skip to Step 2.

If it does **not**, the user has to create a token first (you cannot create it for
them — it's issued in the web portal):

1. Tell the user: open the ArzStack Hub portal → **Perfil → Tokens de CLI** →
   create a token, and copy it (it starts with `arz_` and is shown only once).
2. Then run login. If you know the hub URL, pass it; otherwise the CLI will ask:
   ```bash
   npx @arzstack/hub login --hub https://hub.<org>.cloud --token arz_xxxxx
   ```
   (Or just `npx @arzstack/hub login` and let it prompt for both.)

Prefer letting the user paste the token into the `login` prompt themselves rather
than putting a secret on a command line you log — but `--token` is fine if they
hand it to you.

## Step 2 — Find and install the item

**When the user is vague** ("install the ticket-summary skill", "what MCPs do we
have?", "set me up with our hub stuff") → use the interactive browser. It shows
types with counts, then a paginated list, and handles the project/global choice:

```bash
npx @arzstack/hub install        # or: npx @arzstack/hub browse
```

It will show something like:
```
Instalar — escolha um tipo:
  a. Skill (3)
  b. MCP Server (1)
  c. Knowledge (2)
```
…then a numbered, paginated list of items (`[m] mais` for the next page,
`[v] voltar` to go back), then `Instalar onde? [g] global · [p] projeto`.

Because the browser is interactive, run it so its prompts reach the user (an
inherited terminal), or — if you can't drive an interactive prompt — fall back to
the direct form below by first reading the list with `browse` output and then
installing the chosen id.

**When you already know the item id** (the user named it precisely, or you saw it
in a previous `browse`/web link) → install directly and pass the scope explicitly
so there's no prompt:

```bash
npx @arzstack/hub install <item-id> --global   # or --project
```

**Choosing project vs global** — ask if it's not obvious. Rule of thumb:
- **global** (`~/.agents/skills`): the user wants it available everywhere, in any
  project, across their agents. This is the default and the common case.
- **project** (`.agents/skills`): the item is specific to the repo you're in
  (e.g., a project-scoped MCP or a skill only this codebase needs).

## What each item type does on install

The CLI dispatches by type — explain to the user what's about to happen, since it
runs real commands:
- **skill** → `git clone` of the item's repo into the skills dir. After it lands,
  the skill is available to the agent on the next session/reload.
- **mcp_server** → runs the item's configure command (e.g., `claude mcp add …`).
- **knowledge** → clones/downloads reference material, or prints the link.
- **automation** → prints import instructions (e.g., for n8n); not auto-installed.

The CLI always prints the exact command and destination and asks to confirm
before running (unless `--yes`). Surface that to the user — don't blindly pass
`--yes` unless they asked you to.

## After installing

- Tell the user **what** was installed and **where** (the path the CLI printed).
- For a **skill**, note it becomes active on the next agent session — they may
  need to restart/reload the agent to pick it up.
- For an **MCP server**, confirm whether the configure step succeeded.

## Troubleshooting

- **`Hub respondeu 401`** → the token expired or was revoked. Re-run
  `npx @arzstack/hub login` with a fresh token from Perfil → Tokens de CLI.
- **`Faça login…`** → no saved config; do Step 1.
- **`Já existe: <dest>`** → the item is already installed at that path. Don't
  overwrite blindly — confirm with the user (remove the old one or use `--dir`).
- **`Esta skill não tem repositório git`** → the catalog item has no installable
  source; point the user to "Abrir recurso" / "Ver origem" on the item's page.
- **`npx` not found** → the user needs Node.js 18+ installed.

## Safety

These items are **curated and governed by the user's organization** — but the CLI
still runs `git clone` and configure commands. Always let the user see the command
and destination the CLI prints before it executes, and never silently pass `--yes`
on their behalf unless they explicitly asked for an unattended install. You are
installing org-approved tools, not arbitrary code from the internet — keep it that
way.
