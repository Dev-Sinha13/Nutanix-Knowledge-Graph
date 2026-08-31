---
type: source
tags: [notes]
created: 2026-08-30
updated: 2026-08-30
source_note: "[[inbox/ENG-949861 - Authoring scope mutation guard/01 - Implementation & Decisions]]"
sources:
  - "[[entities/eng-949861]]"
  - "[[concepts/mutation-guard]]"
---

# ENG-949861 implementation (source)

## Summary
Implements a v4 handler-layer guard that returns 403 when a PC caller tries to update or delete an NC-authored Role or ACP. Placed before `AllowOperation` so Lattice `ApplyChange` sync still works and the error is durable if the blanket PC block is later relaxed.

## Key Points
- Condition: request on PC AND stored `authoringScope == "NC"`
- Shared helper `IsAuthoringScopeMutationBlocked`
- Directional NC→PC only; v4 only
- Local tests green; not committed; Tavern deferred (needs NC→PC product-flip)

## Mentioned Pages
- [[entities/eng-949861|ENG-949861]]
- [[entities/eng-915519|ENG-915519]]
- [[entities/iam-themis|iam-themis]]
- [[concepts/mutation-guard|mutation guard]]
- [[concepts/authoring-scope|authoringScope]]
- [[concepts/allow-operation|AllowOperation]]
