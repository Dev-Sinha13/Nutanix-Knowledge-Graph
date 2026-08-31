---
tags: [eng-932537, handlers, derivation]
---

# Handler Logic — derivation chokepoints

Each of the three entity types (Entity, Role, AP) has a different derivation strategy. The chokepoints below are where the work happens; they are the places to touch if behavior needs to change.

## Entities — direct seeding (no derivation)

Entities are the "ground truth" — their `products` are declared per-entity in `objects.json` and copied verbatim through the seeding paths.

### v1 + /proxy path

- **Chokepoint:** `services/server/storage/util.go::FromLoadObjectRequest` (line ~1255)
- **What it does:** `newob.ProductList = ob.Products`
- **Called by:** Both v1 direct (`serveObjectPOST` in `seeding/load_objects.go`) and /proxy-forwarded (`apihandler/object_seeding.go::loadObjects`) routes funnel through `LoadObjectsData` → `generateValidatedObjectMap` → `FromLoadObjectRequest`. Single chokepoint = both surfaces fixed simultaneously.
- **Validation:** `modelutil.Validate(&localObject, strfmt.Default)` runs `Validate(formats)` only — NOT `ContextValidate`. This is what lets clients send `products` despite the swagger model's `readOnly: true` marker on the field. Deliberate (D23 / P10).

### v4 path

- **Chokepoint:** `services/server/storage/util.go::FromV4EntityRequest`
- **What it does:** `newob.ProductList = ob.ProductList`
- **Called by:** v4 `SeedV4Entities` handler
- **Note:** No swagger `readOnly` complication because the v4 model treats `ProductList` as a normal field.

## Roles — union via `ComputeAccessibleEntitiesList`

Role `products` is server-computed: the sorted, deduplicated union of `product_list` across every entity the role's operations act on.

### Chokepoint

- **`services/server/apihandler/util.go::ComputeAccessibleEntitiesList`** — already existed pre-ENG-932537 for the `accessibleEntities` field; extended in this initiative to also collate products via a new helper `getUIDisplayNamesAndProductsForEntities`.
- **Called on EVERY role create AND update.** This is critical: a role's products must be recomputed not just at creation but whenever its `operationList` changes, because the set of touched entities can change.
  - Q-TD1 in [[09 - Open Questions & Followups]] tracks a related gap: server allows updating `operationList` on a role without invoking the full create-time validation chain; needs an `or-recompute-on-operationList-change` audit.

### Determinism

Before writing to storage, the union is `sort.Strings`'d so that:

- The role's etag is deterministic across replicas
- Lattice replication produces byte-stable payloads (replicas don't tag-thrash on rebuild)

## AccessPolicies — OUT OF SCOPE (D24)

The AP wiring (formerly tracked here) was reverted 2026-05-29 per user direction. There is no `validateAP` extension, no `FromAPItoStorageAccessPolicy` ProductList assignment, no `v4_access_policy.go` global-update hook, no /proxy AP transparency check. If reopened, see the reflog SHAs in §17 of the in-repo notes.

## Read-only enforcement summary

| Surface | Enforcement |
|---|---|
| v1 `LoadObjectRequest` | Swagger `readOnly: true` exists but bypassed at runtime because handler calls only `Validate` (not `ContextValidate`). D23. |
| v4 `EntityConfig` | No request-side products complications; field is a normal `[]string`. |
| v4 `Role` (create / update request) | Server unconditionally recomputes `products` from operations → entities; any client-set value would be overwritten. |

## See also

- [[03 - Repository Map & Flow]] for the per-file ownership map
- [[05 - Storage & Migration]] for the SQL writes the handlers ultimately invoke
- [[07 - API Surface Matrix]] for the wired-vs-deferred status of each surface
- [[08 - Decisions Log]] D23, D24 for the read-only / AP-rollback rationale
- Up: [[00 - Index]]
