---
type: concept
created: 2026-08-30
tags: [term]
aliases: [omitempty]
sources:
  - "[[sources/eng-915519-index]]"
  - "[[sources/eng-932537-overview]]"
---

# omit-on-empty

## Definition
Wire/storage convention: empty origin or product values are omitted rather than serialized, so older peers and existing rows remain compatible.

## Key Characteristics
- `authoringScope` uses omit-on-empty with scalar `{NC, PC}`
- `products` uses `omitempty` for forward-compat with older lattice peers
- Existing-row backfill for `authoringScope` is still an open product decision (`''` vs cluster-aware UPDATE)

## Applications
- Rolling federation where not every cluster understands the new fields

## Related Concepts
- [[concepts/authoring-scope|authoringScope]]
- [[concepts/products-field|products]]

## Related Entities
- [[entities/eng-915519|ENG-915519]]
- [[entities/eng-932537|ENG-932537]]

## Mentions in Source
- "scalar enum `{NC, PC}`, server-stamped, immutable, omit-on-empty" — [[sources/eng-915519-index]]
- "Survives lattice replication across clusters (`omitempty` for forward-compat with older peers)" — [[sources/eng-932537-overview]]
