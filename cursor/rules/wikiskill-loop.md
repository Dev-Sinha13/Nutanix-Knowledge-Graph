---
title: Rule — wikiskill-loop
type: cursor-rule
created: 2026-08-31
updated: 2026-08-31
live_path: /Users/dev.sinha/.cursor/rules/wikiskill-loop.mdc
also_copy_to: /Users/dev.sinha/nutanix-core/.cursor/rules/wikiskill-loop.mdc
alwaysApply: true
---

# wikiskill-loop.mdc

User-global always-on rule. Also copy into the git repo so teammates and cloud agents get it. As of 2026-08-31 it is **not** in `nutanix-core/.cursor/rules/`.

Index: [[cursor/00 - Index]]

## File body (`live_path`)

```mdc
---
description: WikiSkill loop — keep raw traces, wiki knowledge, and executable skills separate; compile experience into the vault; load skills at inference, not the whole wiki.
alwaysApply: true
---

# WikiSkill loop

Do not treat chat history as memory. Compile experience into three layers and keep them separate.

Vault: `/Users/dev.sinha/Documents/Obsidian Vault/Nutanix-Core`

| Layer | Where | Role |
| --- | --- | --- |
| Raw | `inbox/` plus in-repo `.notes/` | Immutable sources. SHAs, CI, commands stay in `.notes/`. |
| Wiki | `wiki/` | Compounding knowledge: entities, concepts, `wiki/patterns/`. Never roll this back. |
| Skills | `~/.cursor/skills/` | Short executable procedures. These are what you follow while solving a ticket. |

## While solving (Inference)

1. Load relevant **skills** (`iam-ticket-workflow`, `wikiskill-loop`). Do **not** dump the whole wiki into context.
2. Search the vault only for the few entity/concept/index pages that match the ticket.
3. Re-verify live code and GitHub before acting; vault notes go stale.
4. Do not invent PR numbers, SHAs, or CI status.

## After a real work turn (Wiki Maintainer)

If you implemented, decided, failed, or learned something durable:

1. Patch the ticket `inbox/.../00 - Index.md` and the in-repo `.notes/` file.
2. Add or update a **pattern** under `wiki/patterns/` (failure mode or strategy + workaround). Append evidence; do not rewrite history.
3. Append `wiki/log.md` and, if you changed a skill, `wiki/skill-impact.md` (proposal, accepted/rejected, why).
4. Promote stable facts onto entity/concept pages. Leave narrative in inbox.

## Skill updates (Skill Proposer)

Change a Cursor skill only from accumulated wiki patterns plus this turn’s evidence. One skill per proposal. If a workflow change would make things worse, keep the skill and still keep the wiki pattern (wiki is never rolled back). Prefer general procedures over model-specific hacks.
```
