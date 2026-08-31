---
title: Cursor + Obsidian setup (this machine)
type: setup
created: 2026-08-31
updated: 2026-08-31
live_vault: /Users/dev.sinha/Documents/Obsidian Vault/Nutanix-Core
---

# Cursor + Obsidian setup (this machine)

This is the operational setup, not the talk script. Talk script: [[How to set this up]]. File index: [[cursor/00 - Index]].

## What must be true

1. **One vault open:** `/Users/dev.sinha/Documents/Obsidian Vault/Nutanix-Core`  
   Not the parent `Obsidian Vault`. Plugins and settings are per vault.
2. **Folder contract:** `inbox/` (sources) + `wiki/` (graph) + `cursor/` (these copies).
3. **Node on PATH** so MCP can run `npx` (`brew install node` if `node --version` fails). Cursor’s bundled Node is not enough.
4. **MCP** in [[cursor/mcp]] → `~/.cursor/mcp.json`, then Settings → MCP → toggle on. First run needs network.
5. **User rule** [[cursor/rules/wikiskill-loop]] with `alwaysApply: true`.
6. **User skills** [[cursor/skills/wikiskill-loop]] and [[cursor/skills/iam-ticket-workflow]].
7. **Repo rules** (in `~/nutanix-core`, not this vault): [[cursor/rules/iam-products-field]], [[cursor/rules/iam-tavern-local-k3d]].
8. **User rules** in Cursor Settings: [[cursor/rules/user-rules]] (git, PRs, browser verify) plus [[cursor/rules/gerrit-push-for-review]].
9. Settings → Rules and Skills enabled.

## Obsidian side (once)

1. Create / open **this** vault folder, not a parent.
2. `inbox/` + `wiki/{entities,concepts,sources,schema,patterns}` + `Home.md`.
3. Optional: Community plugins → Karpathy LLM Wiki. Wiki folder = `wiki`. Schema on. Restart after changing wiki folder.
4. Plugin LLM key is optional. Graph view works on wikilinks without it.
5. Query Wiki in Obsidian is a **different** LLM from Cursor. One key does not power the other.

## Cursor side (once)

1. `node --version` in a normal terminal. Install Node if missing.
2. Write `~/.cursor/mcp.json` from [[cursor/mcp]]. Use an **absolute** path to this vault.
3. Restart Cursor. Settings → MCP → obsidian on → trust first run.
4. Verify: agent can list vaults and read `Home.md`.
5. Copy skills into `~/.cursor/skills/<name>/SKILL.md` from [[cursor/skills/wikiskill-loop]] and [[cursor/skills/iam-ticket-workflow]].
6. Copy [[cursor/rules/wikiskill-loop]] to `~/.cursor/rules/wikiskill-loop.mdc`. Also copy into `nutanix-core/.cursor/rules/` so the repo carries the loop.
7. Keep ticket rules in the git repo (products, tavern). Do not fold the Kronos map into those files.

## Daily loop

1. New ticket → `inbox/<ID>/00 - Index.md`
2. Cursor: search a few vault pages → audit code → implement
3. Evidence → repo `.notes/` (SHAs, CI). Do not copy SHAs onto concept pages.
4. Map/status → inbox index
5. Durable failure/strategy → `wiki/patterns/`
6. Promote stable facts → entity/concept pages
7. Change a skill only from patterns + this turn; log [[wiki/skill-impact]]

## New-chat gate

New agent, no bootstrap paste:

> What do you do before planning an IAM ticket?

Pass: vault search, related nodes only, audit code, worktree / VERIFIED vs INFERRED, `.notes/` at the end.  
Fail: starts coding, or asks you to paste context.

## Pitfalls

- Nested vault / wrong window (parent vs `Nutanix-Core`).
- MCP path pointing at the parent vault (current `mcp.json` as of 2026-08-31 — see [[cursor/mcp]]).
- No `npx` → MCP stays dead.
- `@` the whole vault. Walk a few edges instead.
