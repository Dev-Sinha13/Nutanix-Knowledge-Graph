---
type: concept
created: 2026-08-30
tags: [method]
aliases: [FetchShardData, WriteShardData, resolver shard ACP]
sources:
  - "[[sources/eng-910347-index]]"
---

# shard copy

## Definition
Bulk lattice bootstrap path (`FetchShardData` / `WriteShardData`) that copies IAM state onto a newly joined PC. For ACPs, identities must be enriched on the source and resolved on the target rather than copied verbatim.

## Key Characteristics
- Sibling to incremental ApplyChange handling already shipped in iam-themis#1535
- No ntnx-api-iam schema change (internal lattice RPC)
- Collides with OPEN PR #1607 on the same `WriteShardData` block

## Applications
- Fresh PC join rewrites cross-cluster user/group UUIDs to local UUIDs

## Related Concepts
- [[concepts/lattice|lattice]]
- [[concepts/identity-federation|identity federation]]
- [[concepts/global-acp|global ACP]]

## Related Entities
- [[entities/eng-910347|ENG-910347]]
- [[entities/eng-910344|ENG-910344]]
- [[entities/iam-themis|iam-themis]]

## Mentions in Source
- "Extend the enrich-on-source / resolve-on-target ACP identity handling … to the **bulk shard-copy bootstrap** path (`FetchShardData` / `WriteShardData`)" — [[sources/eng-910347-index]]
