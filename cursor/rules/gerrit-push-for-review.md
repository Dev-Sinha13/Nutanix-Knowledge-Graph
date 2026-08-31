---
title: Rule — Gerrit push for review
type: cursor-rule
created: 2026-08-31
updated: 2026-08-31
live_path: Cursor workspace / user rule (not a standalone ~/.cursor/rules file as of 2026-08-31)
alwaysApply: true
---

# Gerrit: code review pushes

Always-on in this workspace. There is no `~/.cursor/rules/*.mdc` file for this yet — it lives as a Cursor workspace/user rule. If you want it as a file, copy the fence into `~/.cursor/rules/gerrit-push-for-review.mdc` and/or `nutanix-core/.cursor/rules/`.

Index: [[cursor/00 - Index]]

## File body

```mdc
---
description: Never push directly to Gerrit; use refs/for for code review
alwaysApply: true
---

# Gerrit: code review pushes

- Do **not** run `git push` straight to a Gerrit branch (e.g. `refs/heads/<branch>`) when the intent is to open or update a change request.
- To raise or update a CR, push to the review ref: `refs/for/<branchname>` (for example `git push <remote> HEAD:refs/for/main` or your target branch).
- Use the project’s documented remote and branch naming; the pattern is always `refs/for/<target-branch>` for review, not a direct branch push.
```
