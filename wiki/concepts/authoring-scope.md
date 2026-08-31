---
type: concept
created: 2026-08-30
tags: [term]
aliases: [authoringScope, authoring_scope]
sources:
  - "[[sources/eng-915519-overview]]"
  - "[[sources/eng-915519-index]]"
  - "[[sources/eng-949861-implementation]]"
---

# authoringScope

## Definition
A read-only field on global IAM Access Control Policies and Roles that records the product cluster on which the entity was originally created. Design locked 2026-06-01 as a **scalar enum** `{NC, PC}`: server-stamped, immutable, omit-on-empty.

## Key Characteristics
- Only needed on lattice-replicated (global) entities
- Clients can read it; they never write it
- UPDATE paths skip the column
- Distinct from [[concepts/products-field|products]]: who authored vs what the entity applies to

## Applications
- Downstream UIs show origin and gate edit permissions
- [[concepts/mutation-guard|Mutation guard]] (ENG-949861) keys PC-side update/delete blocking on stored `authoringScope == "NC"`

## Related Concepts
- [[concepts/products-field|products]]
- [[concepts/kronos|Kronos]]
- [[concepts/lattice|lattice]]
- [[concepts/omit-on-empty|omit-on-empty]]
- [[concepts/mutation-guard|mutation guard]]

## Related Entities
- [[entities/eng-915519|ENG-915519]]
- [[entities/eng-949861|ENG-949861]]
- [[entities/nutanix-central|NC]]
- [[entities/prism-central|PC]]

## Mentions in Source
- "A read-only field on global IAM Access Control Policies (ACPs) and Roles. It records the **product cluster on which the entity was originally created**" — [[sources/eng-915519-overview]]
- "Design locked (2026-06-01): **scalar enum** `{NC, PC}`, server-stamped, immutable, omit-on-empty." — [[sources/eng-915519-index]]
- "`authoringScope` is who-authored / immutable; `products` is what-it-applies-to / server-derived per-entity-type" — [[sources/eng-915519-index]]
