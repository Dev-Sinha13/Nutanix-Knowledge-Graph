---
type: source
tags: [notes]
created: 2026-08-30
updated: 2026-08-30
source_note: "[[inbox/ENG-915519 - Global IAM authoringScope/00 - Index]]"
sources:
  - "[[entities/eng-915519]]"
  - "[[concepts/authoring-scope]]"
  - "[[concepts/kronos]]"
---

# ENG-915519 index (source)

## Summary
Index for the Global IAM `authoringScope` work: a read-only field on global ACPs and Roles recording originating product (NC or PC). Design locked 2026-06-01 as a scalar enum `{NC, PC}`, server-stamped, immutable, omit-on-empty.

## Key Points
- Three open PRs: `ntnx-api-iam#910`, `iam-utils#357`, `iam-themis#1601`
- Hard blocker: go-swagger version pin (Manish)
- Soft open: existing-row backfill
- Distinct from `products` (ENG-932537): who-authored vs what-it-applies-to

## Mentioned Pages
- [[entities/eng-915519|ENG-915519]]
- [[entities/eng-932537|ENG-932537]]
- [[entities/ntnx-api-iam|ntnx-api-iam]]
- [[entities/iam-utils|iam-utils]]
- [[entities/iam-themis|iam-themis]]
- [[entities/manish-lokur|Manish Lokur]]
- [[entities/aditya|Aditya]]
- [[entities/praveen|Praveen]]
- [[concepts/authoring-scope|authoringScope]]
- [[concepts/kronos|Kronos]]
- [[concepts/omit-on-empty|omit-on-empty]]
