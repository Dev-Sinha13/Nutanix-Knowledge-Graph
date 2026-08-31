---
type: source
tags: [notes]
created: 2026-08-30
updated: 2026-08-30
source_note: "[[inbox/ENG-915519 - Global IAM authoringScope/01 - Overview]]"
sources:
  - "[[entities/eng-915519]]"
  - "[[concepts/authoring-scope]]"
---

# ENG-915519 overview (source)

## Summary
Defines `authoringScope` as a read-only, server-stamped, immutable origin field on global ACPs and Roles. Only lattice-replicated (global) entities need it. Explicitly not the same as `products`.

## Key Points
- Records the product cluster where the entity was originally created
- Clients can read it but never write it
- UPDATE statements skip this column
- Array shape was considered for future co-authoring, then reversed to scalar

## Mentioned Pages
- [[entities/eng-915519|ENG-915519]]
- [[entities/nutanix-central|NC]]
- [[entities/prism-central|PC]]
- [[concepts/authoring-scope|authoringScope]]
- [[concepts/products-field|products]]
- [[concepts/lattice|lattice]]
