---
title: Skill — iam-ticket-workflow
type: cursor-skill
created: 2026-08-31
updated: 2026-08-31
live_path: /Users/dev.sinha/.cursor/skills/iam-ticket-workflow/SKILL.md
---

# iam-ticket-workflow / SKILL.md

User skill for Nutanix IAM / Kronos tickets. Keep this short; point at the vault and `.notes/` for the rest. Patterns: [[cursor/skills/iam-ticket-workflow-PURPOSE]].

Index: [[cursor/00 - Index]]

## File body (`live_path`)

```md
---
name: iam-ticket-workflow
description: How to start and close Nutanix IAM / Kronos tickets in iam-themis, ntnx-api-iam, iam-utils, and iam-bootstrap. Use when working an ENG-* IAM issue, Global IAM, authoringScope, products field, lattice, ACP, tavern tests, or multi-repo IAM changes.
---

# IAM ticket workflow

## Start

1. Search the vault (`inbox/` indexes, `wiki/index.md`, related entities/concepts). Do not load every wiki page.
2. Audit code **before** planning: grep vendor models, converters, storage column lists, filter maps. Record what is already wired.
3. Pick repos from the chain; do not assume all six are needed:
   `ntnx-api-iam` → `iam-utils` → `iam-themis` → `iam-bootstrap` → `iam-user-authn` → `iam-deployment`
4. `git rev-parse --abbrev-ref HEAD` before any significant git operation (parallel agents drift branches).

## Question framing

Headline + ticket ID, one paragraph of background, A vs B with consequences, targeted unknowns, what is already decided. No “thoughts?” dumps.

## Review comments

| Kind | Action |
| --- | --- |
| Mechanical (fmt, typo) | Do it |
| Explicit human request, local | Do it if you understand why |
| Design pushback | Ask the design lead |
| Bot-only | Ignore unless a human asked |

## Verification ladder (Go)

`gofmt -l` → `go build -mod=vendor` → `go vet` (new warnings only vs master) → `go test -mod=vendor -count=1` → re-verify **after rebase**. Failures on untouched files: stash and retest on bare HEAD before “fixing.”

## Close the turn

Update the ticket `.notes/` file (decisions, files, gates, changelog). Update the vault inbox index. Add a `wiki/patterns/` page if a failure mode or strategy is new. See `wikiskill-loop`.

## Fragile details

- Subject-only git commits; bodies live in `.notes/` / PR.
- Never `--no-verify`. Strip Cursor `Co-authored-by` with `git commit-tree` if needed.
- `--force-with-lease` only, never plain `--force`.
- ASCII-only Tavern YAML. Do not `gofmt -w` vendor files.
- ENG-932537: Roles + Entities only; AccessPolicies out of scope.

More recipes: [PURPOSE.md](PURPOSE.md)
```
