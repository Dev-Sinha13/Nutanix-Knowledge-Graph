---
type: concept
created: 2026-08-30
tags: [method]
aliases: [IAM Entity Synchronization]
sources:
  - "[[sources/eng-910347-index]]"
  - "[[sources/eng-910350-index]]"
  - "[[sources/eng-915519-overview]]"
  - "[[sources/eng-949861-implementation]]"
---

# lattice

## Definition
The IAM entity synchronization fabric that replicates global entities across clusters. Incremental sync uses ApplyChange; bulk bootstrap uses shard copy (`FetchShardData` / `WriteShardData`).

## Key Characteristics
- Global entities are lattice-replicated; tenant-local entities are not
- Internal Lattice RPCs are not v4 wire fields
- Sync paths must still mutate NC-authored entities on PC even when API callers cannot

## Applications
- Identity UUID enrich-on-source / resolve-on-target for ACPs
- Keeping PC copies of NC-authored Roles/ACPs in sync despite mutation guards on the API layer

## Related Concepts
- [[concepts/shard-copy|shard copy]]
- [[concepts/global-acp|global ACP]]
- [[concepts/mutation-guard|mutation guard]]

## Related Entities
- [[entities/iam-themis|iam-themis]]
- [[entities/eng-910347|ENG-910347]]
- [[entities/eng-910350|ENG-910350]]

## Mentions in Source
- "A \"global\" entity is one that is lattice-replicated across clusters." — [[sources/eng-915519-overview]]
- "so the internal Lattice `ApplyChange` sync — which must keep updating/deleting NC-authored entities on PC to stay in sync — is unaffected." — [[sources/eng-949861-implementation]]
