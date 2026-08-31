---
tags: [eng-932537, api, scope, matrix]
---

# API Surface Matrix

Which API surfaces actually carry `products` today, vs. what was deliberately deferred or removed. The clearest at-a-glance view of scope.

## Legend

- ✅ wired — surfaces the field; tested via unit + tavern
- ⏭ deferred — explicitly out of scope for this initiative; no design doc backs widening
- ❌ removed — was wired in Q-L3 (AP) and reverted 2026-05-29 per D24
- N/A — endpoint does not exist on this surface (e.g., /proxy does not expose a GET for entities)
- 🔀 derived — surfaces value but the value itself is computed server-side, not client-supplied

## Read paths

| Entity type | v1 GET single | v1 LIST | v1 LIST `$filter=products` | v1 LIST `$select=products` | v4 GET single | v4 LIST | v4 `$filter` | v4 `$select` | /proxy GET |
|---|---|---|---|---|---|---|---|---|---|
| Entity (`Object`) | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | N/A |
| Role | ⏭ | ⏭ | ⏭ | ⏭ | ✅ 🔀 | ✅ 🔀 | ✅ | ✅ | N/A |
| AccessPolicy | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |

Notes:

- Entity reads work on every supported surface.
- Role v4 reads were wired in the original Step 3 work and surface the server-derived value. Role v1 widening is deferred (D18). The v1 surface for roles exists but does not carry `products` on the response.
- AccessPolicy is fully out of scope after D24. Was wired (commit `a411a8943` in iam-themis), then reverted via `git rebase --onto 7895a1e8f a411a8943`.
- /proxy has no dedicated GET surface for these entity types; /proxy is a write-forwarding endpoint that dispatches to the v1 underlying endpoints.

## Write paths

| Entity type | v1 POST `/load-objects` | v1 POST `/load-roles` | v1 POST `/load-access-policies` | v4 POST entities (seed) | v4 POST roles | v4 POST APs | /proxy POST `/load-objects` | /proxy POST others |
|---|---|---|---|---|---|---|---|---|
| Entity (`Object`) | ✅ accepts `products` | N/A | N/A | ✅ accepts `ProductList` | N/A | N/A | ✅ accepts `products` (forwards to v1 chokepoint) | N/A |
| Role | N/A | ⏭ ignored (server recomputes) | N/A | N/A | ⏭ ignored (server recomputes) | N/A | N/A | ⏭ |
| AccessPolicy | N/A | N/A | ❌ removed (D24) | N/A | N/A | ❌ removed (D24) | N/A | ❌ |

Notes:

- Entity v1 + /proxy + v4 all accept `products` on the request body. Validation is per-cluster `ProductAllowedTypesMap` at *bootstrap config-load* time only — NOT at runtime in iam-themis (D21).
- Role write paths: server recomputes from accessible entities on every create AND update. Any client-supplied `products` is overwritten without error.
- AP write paths: removed. The v4 `AuthorizationPolicy` schema in `ntnx-api-iam` no longer carries a `products:` block; the Go storage struct has no `ProductList` field; no migration touches the `access_policy` table.

## OData filterability summary

Both entities and roles support `?$filter=products/any(p: p eq 'NC')` on their v4 LIST endpoints. `?$filter=products/all(...)` is not specifically tested but should work via the existing OData parser.

For the v1 LIST entities endpoint, the same filter works — `ObjectSupportedFilterFields` already includes `"products"` (added in §3).

Role v1 LIST does NOT accept the filter — deferred (D18). AP v1 + v4 LIST do NOT accept the filter — removed (D24).

## Bootstrap seeding paths

| Path | Carries products? |
|---|---|
| `iam-bootstrap` v1 path (`loadObjects` in `configload.go`) | ✅ fixed in this initiative; was silently dropping `c.ProductList` |
| `iam-bootstrap` v4 path (`SeedEntitiesV4` in `v4_util.go`) | ✅ fixed in this initiative; was silently dropping `ent.ProductList` |
| `iam-themis` runtime `objects.json` reload | ✅ wired via `FromLoadObjectRequest` |

## What's NOT in this matrix

Things deliberately excluded:

- **Authoring/ownership semantics** — governed by `isGlobal` + `authoringScope` (ENG-915519). A role with `products: ["NC", "PC"]` can still be locally editable on PC if `isGlobal=false`. Do NOT key any read-only logic off `products`.
- **Multi-cluster lattice replication tests** — requires a multi-cluster rig we don't have. The single-cluster tavern tests validate the in-cluster contract; cross-cluster behavior is logged as Q-TD3 in [[09 - Open Questions & Followups]].
- **Runtime `ProductAllowedTypesMap` enforcement** — D21; matches v4 behavior; only the bootstrap config-load path validates.
- **AccessPolicies on any surface** — D24; was wired then reverted.

## See also

- [[02 - Canonical Spec]] for the field-shape contract behind every cell
- [[06 - Handler Logic]] for the code paths each ✅ cell relies on
- [[08 - Decisions Log]] D24 for the AP removal
- [[09 - Open Questions & Followups]] for what the ⏭ cells would entail if/when re-prioritized
- Up: [[00 - Index]]
