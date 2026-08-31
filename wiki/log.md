---
title: Wiki log
type: log
created: 2026-08-30
---

# Wiki log

- **2026-08-30** — Cursor ingest of `inbox/` (ticket indexes + overviews + technical stack). Created 7 source pages, 10 concept pages, 18 entity pages. Plugin Ingest was not used (no LLM provider configured).
- **2026-08-30** — WikiSkill alignment ([arXiv:2608.27454](https://arxiv.org/html/2608.27454)): added `wiki/patterns/`, `wiki/skill-impact.md`; compiled process lessons into Cursor skills `wikiskill-loop` and `iam-ticket-workflow`. Wiki persists independently of skill edits.
- **2026-08-30** — Separate Graphify vault for the four IAM repos (`ntnx-api-iam`, `iam-utils`, `iam-themis`, `iam-bootstrap`): `/Users/dev.sinha/Documents/Obsidian Vault/IAM-Repos-Graph`. Pattern: [[patterns/graphify-focused-corpus]]. Not this wiki.
- **2026-08-31** — Copied live Cursor rules, skills, MCP, and user-rules into `cursor/` for Obsidian editing. Index: [[cursor/00 - Index]]. These vault notes are not what Cursor loads at inference; apply back to `live_path`. Noted gaps: `wikiskill-loop.mdc` not in `nutanix-core/.cursor/rules/`; `mcp.json` still points at the parent vault, not `Nutanix-Core`.
- **2026-08-31** — Added `IMPACT/` as a separate human-readable company-impact section (every Kronos deliverable, heavy detail). GitHub landing: repo `README.md`. Vault: [[IMPACT/README]].
