---
type: pattern
created: 2026-08-30
tags: [strategy]
sources:
  - "[[inbox/ENG-932537 - IAM products field/10 - Process Lessons]]"
---

# Verify pre-existing on bare HEAD

## Failure mode
Treating a test failure or vet warning on an untouched file as caused by this change.

## Workaround
Stash (including untracked), re-run the failing test on clean HEAD, restore. If it still fails, log the command + outcome in `.notes/` and ignore it.

## Evidence
Process lesson 8 in ENG-932537.
