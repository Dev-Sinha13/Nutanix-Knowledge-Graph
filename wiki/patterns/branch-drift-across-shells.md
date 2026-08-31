---
type: pattern
created: 2026-08-30
tags: [failure]
sources:
  - "[[inbox/ENG-915519 - Global IAM authoringScope/11 - Process Lessons]]"
---

# Branch drift across shells

## Failure mode
A parallel agent `git switch`es the shared worktree. Later commands run on the wrong branch.

## Workaround
`git rev-parse --abbrev-ref HEAD` at the start of any significant git operation. Prefer worktrees for parallel agents.

## Evidence
ENG-915519 process lessons — multi-repo gotchas.
