---
type: source
tags: [notes]
created: 2026-08-30
updated: 2026-08-30
source_note: "[[inbox/ENG-932537 - IAM products field/01 - Overview]]"
sources:
  - "[[entities/eng-932537]]"
  - "[[concepts/products-field]]"
---

# ENG-932537 overview (source)

## Summary
Defines `products` as a read-only multi-valued field on IAM Entities and Roles. Entities are seeded from `objects.json`; Roles get a server-computed union. AccessPolicies are out of scope after a 2026-05-29 revert (D24).

## Key Points
- PC clusters: `["PC"]`; NC clusters: `["NC", "NCM"]`
- Not user-settable; not keyed off `isGlobal`
- Distinct from `authoringScope` (applies-to vs who-authored)
- Runtime `ProductAllowedTypesMap` validation in iam-themis is not in scope

## Mentioned Pages
- [[entities/eng-932537|ENG-932537]]
- [[entities/eng-915519|ENG-915519]]
- [[entities/iam-bootstrap|iam-bootstrap]]
- [[concepts/products-field|products]]
- [[concepts/authoring-scope|authoringScope]]
