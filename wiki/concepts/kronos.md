---
type: concept
created: 2026-08-30
tags: [field]
aliases: [Global IAM]
sources:
  - "[[sources/eng-915519-overview]]"
  - "[[sources/nutanix-technical-stack]]"
---

# Kronos

## Definition
The broader Global IAM initiative covering federated, lattice-replicated IAM entities across NC and PC: origin (`authoringScope`), product membership (`products`), identity rewrite on shard copy, and authoring guardrails on global ACPs.

## Key Characteristics
- Multi-repo: ntnx-api-iam, iam-utils, iam-themis, iam-bootstrap
- NC authors global entities; PC consumes / replicates them
- Several tickets share reviewers (Aditya, Praveen) and design lead (Manish Lokur)

## Applications
- Distinguishing who authored a global entity vs which products it applies to
- Preventing local identities and PC-side mutation of NC-authored objects

## Related Concepts
- [[concepts/authoring-scope|authoringScope]]
- [[concepts/products-field|products]]
- [[concepts/lattice|lattice]]
- [[concepts/global-acp|global ACP]]

## Related Entities
- [[entities/eng-915519|ENG-915519]]
- [[entities/eng-932537|ENG-932537]]
- [[entities/eng-949861|ENG-949861]]
- [[entities/eng-910347|ENG-910347]]
- [[entities/eng-910350|ENG-910350]]
- [[entities/manish-lokur|Manish Lokur]]

## Mentions in Source
- "Part of the broader **Kronos** / Global IAM initiative" — [[sources/eng-915519-overview]]
