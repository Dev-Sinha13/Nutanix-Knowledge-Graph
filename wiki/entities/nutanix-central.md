---
type: entity
created: 2026-08-30
tags: [product]
aliases: [NC, Nutanix Central, Nutanix Cloud Manager]
sources:
  - "[[sources/eng-915519-overview]]"
  - "[[sources/eng-932537-overview]]"
  - "[[sources/nutanix-technical-stack]]"
---

# Nutanix Central

## Description
Leader product in Global IAM. NC authors global Roles and ACPs; PC consumes them over lattice. Notes disagree on the expansion: the technical stack calls NC "Nutanix Central"; ENG-915519 overview lists `NC` as "Nutanix Cloud Manager". Both names are preserved as aliases. NC clusters allow `products` values `NC` and `NCM`.

## Related Entities
- [[entities/prism-central|Prism Central]]
- [[entities/ncm|NCM]]
- [[entities/eng-915519|ENG-915519]]
- [[entities/eng-949861|ENG-949861]]

## Related Concepts
- [[concepts/authoring-scope|authoringScope]]
- [[concepts/products-field|products]]
- [[concepts/kronos|Kronos]]

## Mentions in Source
- "`NC` — Nutanix Cloud Manager" — [[sources/eng-915519-overview]]
- "Nutanix Central(NC) - Acts as a global management platform over all clusters and deployments" — [[sources/nutanix-technical-stack]]
- "NC clusters | `[\"NC\", \"NCM\"]`" — [[sources/eng-932537-overview]]
