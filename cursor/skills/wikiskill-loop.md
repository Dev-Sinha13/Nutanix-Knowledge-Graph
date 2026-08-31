---
title: Skill — wikiskill-loop
type: cursor-skill
created: 2026-08-31
updated: 2026-08-31
live_path: /Users/dev.sinha/.cursor/skills/wikiskill-loop/SKILL.md
---

# wikiskill-loop / SKILL.md

User skill. YAML `name` + `description` (what **and** when). Do not set disable-model-invocation if you want auto-trigger.

Index: [[cursor/00 - Index]] · rule: [[cursor/rules/wikiskill-loop]]

## File body (`live_path`)

```md
---
name: wikiskill-loop
description: Compiles agent experience into a persistent Obsidian wiki and Cursor skills (WikiSkill). Use at session start on Nutanix IAM tickets, after a work turn that produced a decision or failure, or when the user mentions the vault, wiki, skills, or improving the agent from experience.
---

# WikiSkill loop

Three layers. Do not mix them.

- **Raw:** `inbox/` and repo `.notes/` — sources of truth for evidence
- **Wiki:** `/Users/dev.sinha/Documents/Obsidian Vault/Nutanix-Core/wiki/` — compounding knowledge
- **Skills:** `~/.cursor/skills/` — procedures to execute

## Inference (doing the task)

Follow **skills**, not the whole wiki. Read `wiki/index.md` plus at most a handful of matching pages. Re-verify code/GitHub. Continue with `iam-ticket-workflow` for IAM tickets.

## Maintain (end of a real turn)

1. Update inbox index + `.notes/`
2. Create or patch `wiki/patterns/<slug>.md` with failure/strategy + workaround + evidence
3. Append `wiki/log.md`
4. If a skill changed: append `wiki/skill-impact.md` with accepted/rejected

Wiki pages persist even if a skill edit is reverted.

## Propose skill edits

Only from patterns + this turn. One skill, patch-based, general procedures. Link `PURPOSE.md` to the motivating patterns.
```
