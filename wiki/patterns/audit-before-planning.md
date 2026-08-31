---
type: pattern
created: 2026-08-30
tags: [strategy]
sources:
  - "[[inbox/ENG-932537 - IAM products field/10 - Process Lessons]]"
---

# Audit before planning

## Failure mode
Drafting a multi-step plan to add capability X without grepping current wiring. Plans balloon with vendor edits and handler work that already landed.

## Workaround
1. Grep the field in vendor models
2. Read converters on the related struct
3. Check column lists / field maps in `storage/util.go`
4. Check supported-filter constants
5. Then plan, and state what is already done

## Evidence
ENG-932537 §16 shrank from 9 steps to 2 line edits after the turn-start audit (D20, D23).
