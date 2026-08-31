---
type: concept
created: 2026-08-30
tags: [method]
aliases: [authoring scope mutation guard, IsAuthoringScopeMutationBlocked]
sources:
  - "[[sources/eng-949861-implementation]]"
---

# mutation guard

## Definition
API-handler check that rejects user-initiated update/delete of an NC-authored Role or ACP when the request runs on PC. Returns 403 with "This entity is managed by Nutanix Central and cannot be modified or deleted locally on Prism Central."

## Key Characteristics
- Runs **before** [[concepts/allow-operation|AllowOperation]] so it is not dead code
- Does not apply to Lattice ApplyChange (internal sync still mutates)
- Directional: NC→PC only; v4 only
- Keys on stored scalar `authoringScope == "NC"` plus `productutil.IsPCProduct()`

## Applications
- Clearer error than the existing 400 global-on-PC block
- Remains correct if `AllowOperation`'s blanket PC block is later relaxed for Global IAM GA

## Related Concepts
- [[concepts/authoring-scope|authoringScope]]
- [[concepts/allow-operation|AllowOperation]]
- [[concepts/lattice|lattice]]

## Related Entities
- [[entities/eng-949861|ENG-949861]]
- [[entities/eng-915519|ENG-915519]]
- [[entities/iam-themis|iam-themis]]

## Mentions in Source
- "Block a mutation (**PUT / DELETE**) when **both**: (1) request runs on a **PC** cluster, and (2) the existing entity's `authoringScope` is **NC**." — [[sources/eng-949861-implementation]]
- "Enforce at the **API handler layer** (not persistence), so the internal Lattice `ApplyChange` sync … is unaffected." — [[sources/eng-949861-implementation]]
