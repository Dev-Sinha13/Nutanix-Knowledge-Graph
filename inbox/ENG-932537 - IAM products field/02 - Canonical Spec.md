---
tags: [eng-932537, spec, design]
status: stable
---

# Canonical Spec — the field contract

Single source of truth for the `products` field. All 4 repos MUST match this.

## Field shape

| Property | Value | Notes |
|---|---|---|
| Field name | `products` (plural) | Wire name on all API surfaces (v1 + v4 + /proxy) |
| Go identifier | `ProductList` on storage + request structs; `Products` on response models | Convention mismatch is historical; both serialize to `"products"` on the wire |
| Go type | `[]string` | |
| Postgres column | `product_list text[] DEFAULT '{}'` | Additive idempotent migration; see [[05 - Storage & Migration]] |
| SQLite test mirror | `product_list text NULL` | Because SQLite has no native array type; the storage scanner handles both |
| JSON tag | `json:"products,omitempty"` | `omitempty` is critical for Lattice forward-compat with older peers that don't recognize the field |
| Read-only on API | yes | Server-computed or seeded; client requests cannot set it (or are ignored) |
| Required | no | Empty array `'{}'` is valid and means "not declared" |
| Filterable (v4 OData) | yes | `$filter=products/any(p: p eq 'NC')` works on entities, roles, APs |
| `$select` projection | yes | `?$select=products,displayName` returns only those fields |
| Lattice-replicated | depends on entity type (see below) | |

## ProductAllowedTypesMap

Defined in `iam-utils/configutil/config_load_util.go`. Keyed by the cluster's running product type:

```go
var ProductAllowedTypesMap = map[string][]string{
    "PC": {"PC"},
    "NC": {"NC", "NCM"},
}
```

**Where it's enforced:** ONLY at bootstrap config-load time, via `iam-utils/configutil::isConfigValid`. NOT enforced at iam-themis runtime on the load-objects path (D21).

**Implications:**
- A bootstrap with an `objects.json` entry containing `productList: ["AHV"]` will fail at startup.
- A runtime POST to `/api/iam/authz/v1/client/load-objects` with the same invalid value will be persisted as-is. Trust boundary: the API caller is assumed to be a cluster-internal service that has already validated.
- If we ever need runtime validation, see P10 in [[09 - Open Questions & Followups]].

## Per-entity-type semantics

### Entities (`storage.Object`)

- **Value source:** Declared per-entity in `iam-themis/services/config/templates/objects/objects.json` as `productList: [...]`
- **Flow:** `ConfigObject` → `LoadObjectRequest` (v1 path) OR `EntityConfig` (v4 path) → `storage.Object.ProductList` → `object.product_list`
- **Returned via:** `models.ClientObject.Products` (v1) and `iam.v4.authz.Entity.products` (v4)
- **Lattice-replicated:** NO. Each cluster seeds the same `objects.json` separately; entities are not lattice-synced.

### Roles (`storage.Role`)

- **Value source:** Server-derived. Union of `object.product_list` across every entity the role's operations act on.
- **Computed by:** `services/server/apihandler/util.go::ComputeAccessibleEntitiesList` (which now also collates products via `getUIDisplayNamesAndProductsForEntities`)
- **When recomputed:** EVERY role create AND update — not just create. This is the chokepoint that keeps `role.products` in sync as entity-side `product_list` evolves.
- **Determinism:** `sort.Strings` before storage so etag is stable and lattice replication is byte-stable.
- **Lattice-replicated:** YES for global roles (via the storage struct's `json:"products,omitempty"` tag).

### AccessPolicies — OUT OF SCOPE (D24)

The AP wiring (formerly tracked here as Q-L3 / D9–D17) was reverted 2026-05-29 per user direction. There is no `storage.AccessPolicy.ProductList` field, no `access_policy.product_list` column, no v4 AP schema entry, and no handler logic. The reflog carries the prior implementation if it's ever reopened (see §17 in the in-repo notes).

## Mutability rules

- **API:** `products` is read-only for users. Do not generate setters or accept the field in request payloads. The v1 `LoadObjectRequest` model has `Products []string` (with `readOnly: true` in the swagger) but the iam-themis runtime handler bypasses `ContextValidate` (where ReadOnly is enforced) by calling only `Validate(formats)`. This intentional bypass enables bootstrap to forward products via the v1 path; see D23.
- **Lattice:** `omitempty` on the storage struct JSON tag so older peers ignore the field.

## See also

- [[03 - Repository Map & Flow]] for where each piece of the spec lives in code
- [[05 - Storage & Migration]] for the Postgres-side contract
- [[06 - Handler Logic]] for the per-entity-type derivation code paths
- [[07 - API Surface Matrix]] for which APIs surface this today
- Up: [[00 - Index]]
