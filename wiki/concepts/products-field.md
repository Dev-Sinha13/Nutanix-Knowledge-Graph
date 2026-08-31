---
type: concept
created: 2026-08-30
tags: [term]
aliases: [products]
sources:
  - "[[sources/eng-932537-overview]]"
---

# products field

## Definition
A read-only multi-valued field (`[]string` / Postgres `text[]`) on IAM Entities and Roles. Entities get product context seeded from `objects.json`; Roles get a server-computed union of products across entities their operations act on.

## Key Characteristics
- PC clusters allow `["PC"]`; NC clusters allow `["NC", "NCM"]`
- Not user-settable
- Not keyed off `isGlobal`
- AccessPolicies are out of scope (reverted 2026-05-29, D24)

## Applications
- Nutanix Central UI filters federated IAM entities by originating product
- Survives lattice replication (`omitempty` for older peers)

## Related Concepts
- [[concepts/authoring-scope|authoringScope]]
- [[concepts/kronos|Kronos]]
- [[concepts/lattice|lattice]]

## Related Entities
- [[entities/eng-932537|ENG-932537]]
- [[entities/eng-915519|ENG-915519]]
- [[entities/nutanix-central|NC]]
- [[entities/prism-central|PC]]
- [[entities/ncm|NCM]]

## Mentions in Source
- "`products` is a **read-only multi-valued field** (Go `[]string`, Postgres `text[]`) on two IAM entity types: **Entities** … and **Roles**" — [[sources/eng-932537-overview]]
- "`authoringScope` records *who authored* an entity … `products` records *which products the entity applies to*" — [[sources/eng-932537-overview]]
