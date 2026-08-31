---
type: entity
created: 2026-08-30
tags: [product]
aliases: [PC, Prism]
sources:
  - "[[sources/eng-915519-overview]]"
  - "[[sources/eng-949861-implementation]]"
  - "[[sources/nutanix-technical-stack]]"
---

# Prism Central

## Description
Cluster product that consumes global IAM entities. `authoringScope` value `PC` means the entity was created on Prism Central. On PC, API callers cannot mutate NC-authored Roles/ACPs; lattice sync still can. PC clusters allow `products: ["PC"]`.

## Related Entities
- [[entities/nutanix-central|Nutanix Central]]
- [[entities/eng-949861|ENG-949861]]
- [[entities/eng-910347|ENG-910347]]

## Related Concepts
- [[concepts/authoring-scope|authoringScope]]
- [[concepts/mutation-guard|mutation guard]]
- [[concepts/shard-copy|shard copy]]

## Mentions in Source
- "`PC` — Prism Central" — [[sources/eng-915519-overview]]
- "Prism acts as the management interface" — [[sources/nutanix-technical-stack]]
