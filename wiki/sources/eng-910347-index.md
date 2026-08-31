---
type: source
tags: [notes]
created: 2026-08-30
updated: 2026-08-30
source_note: "[[inbox/ENG-910347 - Resolver Shard ACP/00 - Index]]"
sources:
  - "[[entities/eng-910347]]"
  - "[[concepts/shard-copy]]"
---

# ENG-910347 index (source)

## Summary
Extends enrich-on-source / resolve-on-target ACP identity handling from incremental ApplyChange (PR #1535) to bulk shard-copy bootstrap (`FetchShardData` / `WriteShardData`) so a newly joined PC rewrites cross-cluster user/group UUIDs locally.

## Key Points
- iam-themis only; no schema change
- Implemented and unit-tested, not committed
- Parent incremental path merged in iam-themis#1535
- Collision with OPEN PR #1607 on the same `WriteShardData` block

## Mentioned Pages
- [[entities/eng-910347|ENG-910347]]
- [[entities/eng-910344|ENG-910344]]
- [[entities/iam-themis|iam-themis]]
- [[concepts/shard-copy|shard copy]]
- [[concepts/lattice|lattice]]
- [[concepts/global-acp|global ACP]]
