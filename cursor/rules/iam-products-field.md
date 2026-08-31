---
title: Rule — iam-products-field
type: cursor-rule
created: 2026-08-31
updated: 2026-08-31
live_path: /Users/dev.sinha/nutanix-core/.cursor/rules/iam-products-field.mdc
alwaysApply: true
ticket: ENG-932537
---

# iam-products-field.mdc

Repo rule on `nutanix-core`. Ticket contract for ENG-932537 (Roles + Entities only; AccessPolicies out of scope). Not a substitute for the Kronos map.

Index: [[cursor/00 - Index]] · concept: [[wiki/concepts/products-field]]

## File body (`live_path`)

```mdc
---
description: Guidance for implementing the 'products' field (ENG-932537) in IAM Roles and Entities. AccessPolicies are out of scope.
globs: iam-themis/services/server/apihandler/*.go, iam-themis/services/server/apiutil/*.go, iam-themis/services/server/storage/**/*.go, iam-themis/services/config/templates/objects/objects.json, iam-utils/configutil/*.go, iam-bootstrap/services/themis-bootstrap/**/*.go, iam-themis/.notes/PRODUCTS_FIELD_CONTEXT.md
alwaysApply: true
---

# ENG-932537: 'products' Field Implementation

> **Scope (revised 2026-05-29):** Roles and Entities only. AccessPolicies are explicitly **out of scope** for this initiative. The Q-L3 AP wiring (§15) was reverted in iam-themis (`a411a8943` dropped) and ntnx-api-iam (`658bbe23` amended to `01fb9033`). See D24 in `iam-themis/.notes/PRODUCTS_FIELD_CONTEXT.md`.

## 1. Data Model & Naming

- **Field name**: `products` (plural). Go type `[]string`, Postgres column `text[]`.
- **Go identifier**: `ProductList` on storage and request structs; JSON tag `"products,omitempty"`.
- **Valid values**: only what `configutil.ProductAllowedTypesMap` allows — keyed by cluster product type:
  - On PC clusters: `["PC"]`
  - On NC clusters: `["NC", "NCM"]`
- **Not valid**: `AHV`, `CALM`, `UNKNOWN`, `Xi`. `Xi` exists as a `productutil` constant for runtime product type but is **not** in `ProductAllowedTypesMap`, so it cannot be seeded as a `productList` entry under current validation.
- **Default**: empty array `'{}'` in Postgres. Treat empty as "not declared" — do not substitute a fallback value at read time.

## 2. Logic by Entity Type

- **Entities (`storage.Object`)**: Value is declared per-entity in `objects.json` as `productList`. Flows: `ConfigObject` → `LoadObjectRequest`/`EntityConfig` → `storage.Object` → `object.product_list`. **Surfaced on both v1 and v4 read paths**: `models.ClientObject.Products` for v1 `GET /api/iam/authz/v1/objects[/{id}]` (built by `storage.ToGetObjectResponse` / `ToListObjectResponse`) and `iam.v4.authz.Entity.products` for v4 `GET /api/iam/v4.0/authz/entities[/{extId}]`. **Accepted on the v1 + /proxy write path**: `models.LoadObjectRequest.ProductList` for direct `POST /api/iam/authz/v1/client/load-objects` and proxy-forwarded `POST /proxy` → `loadObjects`. Validated against `configutil.ProductAllowedTypesMap` at config-load time only (no runtime validation in iam-themis; D21). **Not lattice-replicated** — seeded identically per cluster.
- **Roles (`storage.Role`)**: Server-derived. Union of `object.product_list` across every entity the role's operations act on. Recompute on **every** role create AND update via `ComputeAccessibleEntitiesList` in `services/server/apihandler/util.go` — not just on create. Surfaced on v4 `iam.v4.authz.Role.products` only (v1/proxy widening for roles deferred to a follow-up ticket). Lattice-replicated for global roles via the storage struct's JSON tag.
- **Access Policies**: **OUT OF SCOPE.** No storage column, no handler logic, no request/response field, no schema in `ntnx-api-iam`. The Q-L3 wiring was reverted (D24). Do NOT add `ProductList` to `storage.AccessPolicy`, do NOT add `product_list` to `access_policy` table, do NOT add `products:` to any AP yaml schema. If a future ticket reopens this, the prior implementation is reachable via reflog (`a411a8943` on iam-themis, `658bbe23` on ntnx-api-iam).

## 3. Storage & OData

- **Migration**: `ALTER TABLE <t> ADD COLUMN IF NOT EXISTS product_list text[] DEFAULT '{}';` — additive, idempotent. Existing rows get `'{}'`, not NULL. Apply to `object` and `role` tables only — NOT `access_policy`.
- **OData filterability requires two edits**:
  1. EDM binding in `vendor/.../models/edm/iam/v4/authz/authz_model.go` (sets `IsFilterable`).
  2. Go-side allow-list in `iam-themis/services/server/storage/util.go` — add `products` to `ObjectSupportedFilterFields` (and the role equivalent).
- **`objects.json` is seed data**, not OData metadata. Do not register filterable fields there.

## 4. Mutability & Replication

- **API**: `products` is **read-only** for users. Do not generate setters or accept the field in request payloads.
- **Authoring/read-only enforcement**: governed by `isGlobal` + `authoringScope` (ENG-915519), **not** by `products`. A role with `products: ["NC", "PC"]` can still be locally editable on PC if `isGlobal=false`. Do not key any read-only logic off `products`.
- **Lattice**: use `omitempty` on the storage struct JSON tag so older peers ignore unknown field.

## 5. Bootstrap Seeding (was broken pre-PR; now fixed)

`iam-bootstrap` was dropping `ProductList` when converting `ConfigObject` to `LoadObjectRequest`. Fixed in iam-bootstrap PR #768 (`c87e939e`) on **both** paths below:

- `iam-bootstrap/services/themis-bootstrap/configload/configload.go::loadObjects` (v1 path) — sets `ProductList: c.ProductList`.
- `iam-bootstrap/services/themis-bootstrap/util/v4_util.go::SeedEntitiesV4` (v4 path) — sets `entity.ProductList = ent.ProductList`.

## 6. Reference Files

- `iam-themis/services/server/apiutil/object_util.go` — `ToGetV4ObjectResponse` (entity field projection)
- `iam-themis/services/server/apiutil/role_util.go` — `ToGetV4RoleResponse` (role field projection)
- `iam-themis/services/server/apihandler/util.go` — `ComputeAccessibleEntitiesList`, `getUIDisplayNamesAndProductsForEntities` (role collation)
- `iam-themis/services/server/storage/storage.go` — `Object`, `Role` struct definitions (no `AccessPolicy.ProductList`; out of scope)
- `iam-themis/services/server/storage/util.go` — column lists, field maps, filter allow-lists for object + role only
- `iam-themis/services/server/storage/sql/migrate.go` — the additive migration (object + role tables only)
- `iam-utils/configutil/config_load_util.go` — `ConfigObject`, `ConfigRole`, `ProductAllowedTypesMap` (`ConfigACP.ProductList` exists in the type but is unused in iam-themis runtime; harmless)
- `vendor/github.com/nutanix-core/ntnx-api-iam/iam-server-codegen/models/iam/v4/authz/authz_model.go` — `Entity`, `Role` (NOT `AuthorizationPolicy`; that field was removed from #914)

## 7. Notes & Logging Discipline

The canonical work log for ENG-932537 lives at:

  `iam-themis/.notes/PRODUCTS_FIELD_CONTEXT.md`

Treat this file as part of the deliverable, not an afterthought. Every sustained work turn on this initiative MUST end with that file updated. Specifically, you must add or extend:

1. **The appropriate Step execution log** (§N — currently §3 iam-themis, §14 iam-bootstrap; add new §N sections for new repos) with sub-step status changes, commit SHAs, and gate evidence.
2. **A "Decisions log" row** (D-numbered) for any non-obvious decision — rejected approaches, deferrals, naming reconciliations, vendor strategy choices, git workflow exceptions. One sentence on **why** suffices.
3. **A "Files touched" entry** for every file in every commit you create (path, kind, why).
4. **A "Final gate evidence" block** mirroring existing §3.10b / §4.7 style whenever you run build/test gates — capture actual commands and outcomes.
5. **A Changelog row** at the bottom of the file with date (`YYYY-MM-DD`), session label (`agent session N`), and a 2-4 sentence summary of the turn.
6. **A "PR description draft" subsection** persisting any PR body you compose — PR bodies are otherwise lost on chat reset.
7. **Any reproducibility artifacts**: intermediate commit SHAs from plumbing rewrites, reflog excerpts after force operations, exact test-suite timings, hook-injected trailer behavior, etc. If a future agent would need it to reproduce the turn, log it.

**Before ending any turn that touched any iam-* repo, re-read the relevant §N section and verify your changes from this turn are captured.** If they're not, update before stopping. When in doubt, log more rather than less — the file is the durable memory across chat resets.

Acceptable to skip ONLY for: pure read/exploration turns where no code, commit, or decision was produced.

If the user asks "did you update the notes?" or "what have you logged?", that's a signal you almost certainly under-logged — open the file, audit your turn's actions against what's there, and patch the gaps before answering.
```
