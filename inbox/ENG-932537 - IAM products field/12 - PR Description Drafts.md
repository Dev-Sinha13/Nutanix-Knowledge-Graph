---
tags: [eng-932537, pr-bodies, drafts]
status: needs-backfill
---

# PR Description Drafts

Parking lot for the 5 PR bodies. All 4 existing PRs (#914, #358, #1603, #768) still have GitHub template placeholders; this file has the canonical bodies to copy-paste in. The new iam-utils swagger PR has its body here too.

**Order below = recommended copy-paste order, matching the merge ordering in [[04 - PR Status]].**

---

## 1. iam-bootstrap #768 (FULLY DRAFTED, copy-paste ready)

For `https://github.com/nutanix-core/iam-bootstrap/pull/768`.

### What has been implemented?

Propagates the per-entity `productList` field from `objects.json` all the way through bootstrap seeding into themis on **both** the v1 and v4 paths, so the read-only `products` array on the v4 entity GET/LIST APIs has a value to return. Without this PR, themis (post `ENG-932537` merge) would persist empty arrays and `$filter=products/...` would return nothing.

**Code changes:**

- **v1 seeding path** (`services/themis-bootstrap/configload/configload.go`): extracted `toLoadObjectRequest(c util.ConfigObject) models.LoadObjectRequest` helper and wired `Products: c.ProductList` into the request. Field flows `ConfigObject.ProductList` (json `"productList"`) -> `LoadObjectRequest.Products` (json `"products"`) -> themis `object.product_list` (`text[]`).
- **v4 seeding path** (`services/themis-bootstrap/util/v4_util.go`): extracted `buildV4EntityConfig(ent *models.LoadObjectRequest) *authzV4Model.EntityConfig` helper and wired `entity.ProductList = ent.Products`. The v4 `SeedConfig` request then persists onto the same `object.product_list` column on the themis side.
- **Vendor swap** (TEMP HACK ENG-932537, removed after ntnx-api-iam #914 / iam-utils regen lands):
  - `vendor/github.com/nutanix-core/iam-utils/themisutil/generated/models/load_object_request.go` — copied byte-for-byte from the iam-themis vendor so both repos consume an identical iam-utils model.
  - `vendor/github.com/nutanix-core/ntnx-api-golang-sdk-internal/iam-go-client/v17/models/iam/v4/authz/authz_model.go` — added 9 lines on `type EntityConfig` for `ProductList []string`.

Both helpers are pure functions, so the field-mapping logic is unit-testable without filesystem fixtures or a live `ApiClient`.

**Cross-PR chain (please review in order):**
1. `ntnx-api-iam` #914 — adds `products` to the v4 Entity/Role schemas
2. `iam-utils` #358 — configutil + validation types
3. `iam-themis` #1603 — storage, SQL, API handlers, role collation, OData filter allow-list
4. **This PR** (`iam-bootstrap`) — the seeding glue

The TEMP HACK vendor markers go away with a real revendor once #1 and #2 land.

> Note: there is a pre-existing branch `origin/ENG-932537` (`8a9738ca` "seed products list") covering the same scope. It was deliberately superseded by this branch — it bundled unrelated changes (Dockerfile artifactory URL, go.mod downgrades), used a side-band `objectNameToProducts` map workaround, and shipped tests that inlined production logic. That branch should be closed when this one merges.

### Type of Change

- [x] New feature or improvement (non-breaking change).

### How Has This Been Tested?

Strategy: unit-test the two extracted helpers directly (pure functions over wire models, high signal); rely on the iam-themis tavern e2e tests for cross-component round-trip on seeded clusters.

Scenarios covered by new unit tests:
- `TestToLoadObjectRequest_PropagatesProducts` — v1 happy path with full field round-trip
- `TestToLoadObjectRequest_NoProducts` — legacy entry with no `productList` stays nil/empty
- `TestBuildV4EntityConfig_PropagatesProducts` — v4 happy path
- `TestBuildV4EntityConfig_EmptyProducts` — nil-safe
- `TestBuildV4EntityConfig_SingleProduct` — per-cluster single-element case

Local gates all green: gofmt clean, go vet clean, go build OK, go test all packages OK.

Boxes:
- [x] Unit tests added or updated.
- [ ] API tests added or updated.
- [x] Performance tests considered. *(N/A — additive field, zero new DB queries.)*
- [x] Manual testing performed.

### Checklist

- [x] Self-review completed.
- [x] Code adheres to project style guidelines.
- [x] Relevant comments added for clarity.
- [x] Code meets quality gate criteria.
- [ ] Vulnerability scans show no critical issues. *(Confirm in CI scan run.)*
- [ ] ReadMe and related docs updated as applicable. *(N/A.)*

---

## 2. iam-utils #358 (OUTLINE — needs filling out)

For `https://github.com/nutanix-core/iam-utils/pull/358`.

### What has been implemented?

Adds the configutil + validation types that all downstream IAM repos use for the `products` field:

- `ConfigObject.ProductList []string` (json: `productList`) for entity declarations in `objects.json`
- `ConfigRole.ProductList []string` for role declarations
- `ConfigACP.ProductList []string` for access-policy declarations
- `ProductAllowedTypesMap` (`{"PC": ["PC"], "NC": ["NC", "NCM"]}`)
- `isConfigValid` extension to validate per-entity products against the allow-map at config-load time

Upstream source for the shared field type + validation. Downstream consumers pick this up via standard `go.mod` revendor.

### Type of Change

- [x] New feature or improvement (non-breaking change).

### How Has This Been Tested?

Unit tests on `isConfigValid` for valid PC/NC entries, invalid AHV/CALM/Xi/UNKNOWN, empty list. Round-trip JSON marshal/unmarshal on `ConfigObject` / `ConfigRole` / `ConfigACP`.

Boxes:
- [x] Unit tests added or updated.
- [ ] API tests added or updated. *(N/A for utility library.)*
- [x] Performance tests considered. *(N/A — O(n) over the products list, called once per entity at startup.)*
- [x] Manual testing performed.

### Checklist

Standard 6 boxes; all addressable.

---

## 3. iam-utils ClientObject swagger PR (NEW — not yet opened; fully drafted)

Suggested branch: `add-products-to-client-object`.
Suggested PR title: `themis.yaml: surface products on ClientObject response for v1 GET /objects[/id]`.

### Schema snippet to add

Inside the `ClientObject` schema (sibling to `serviceName`, `tenantName`, etc.):

```yaml
products:
  type: array
  description: >
    List of products this entity belongs to (for example, PC, NC, NCM). Sourced from
    the entity configuration at bootstrap seeding time. Read-only on the v1 API.
  items:
    type: string
  readOnly: true
  minItems: 0
  maxItems: 10
```

### PR body

#### What has been implemented?

Adds the `products` field to the `ClientObject` swagger schema so that the iam-themis v1 `GET /api/iam/authz/v1/objects[/{id}]` and `LIST /objects` response payloads carry the per-entity product context (PC / NC / NCM). Mirrors the existing `products` field shape on `LoadObjectRequest` and on the v4 `Entity` schema; no behavior change in iam-utils itself, just a schema/codegen surface addition.

This is the canonical-iam-utils side of the iam-themis ENG-932537 section-16 change. The downstream iam-themis vendor file currently carries a manual addition of the same `Products []string` field on the generated `models.ClientObject` struct so that iam-themis can ship without blocking on this PR; once this merges and iam-themis revendors, the manual edit becomes a no-op overwrite.

Related: ntnx-api-iam #914 (v4 product schema + AP), iam-utils #358 (configutil), iam-themis #1603, iam-bootstrap #768.

#### Type of Change

- [x] New feature or improvement (non-breaking change).
- [x] Documentation update or release notes required.

#### How Has This Been Tested?

Codegen-only change. Validation = `mvn` regen of go/python/java/sphinx clients, verify generated `models.ClientObject` carries a `Products []string` field with `json:"products,omitempty"`. Wire-level correctness exercised by the iam-themis tavern test `api_tests/objects/test_object_products_v1_and_proxy.tavern.yaml`.

- [ ] Unit tests added or updated.
- [x] API tests added or updated (downstream, in iam-themis #1603).
- [ ] Performance tests considered.
- [x] Manual testing performed (downstream tavern).

#### Checklist

Standard 6 boxes; all addressable.

---

## 4. ntnx-api-iam #914 (OUTLINE — needs filling out)

For `https://github.com/nutanix-core/ntnx-api-iam/pull/914`.

### What has been implemented?

Adds the `products` field to two v4 IAM authz schemas + their EDM filterability bindings:

- `iam.v4.authz.Entity` — read-only multi-valued field; seeded per cluster from `objects.json`
- `iam.v4.authz.Role` — server-computed union of `products` across the role's accessible entities

EDM bindings make both filterable via `?$filter=products/any(...)` and selectable via `?$select=products`. Upstream schema source; downstream consumers (iam-themis vendors `iam-server-codegen`) revendor to pick up the field.

> **Scope note:** `AuthorizationPolicy.products` was originally part of this PR (commit `658bbe23`) but was removed per user direction on 2026-05-29 (D24). The amended commit is `01fb9033` and contains only Entity + Role schema changes. AP yaml file restored to parent state; `accessPolicyProductsDesc` block removed from `iamDefsDescriptions.yaml`.

### Type of Change

- [x] New feature or improvement (non-breaking change).
- [x] Documentation update or release notes required.

### How Has This Been Tested?

- Schema lint (`mvn lint-checker-maven-plugins:generate-report`) clean for both schemas
- Codegen (`mvn install`) produces clean Go/Python/Java/Sphinx clients with `Products []string` on each model
- EDM filterability verified by inspecting the generated `iam-server-codegen` model file and checking `IsFilterable: true`

### Checklist

Standard 6 boxes; all addressable.

---

## 5. iam-themis #1603 (TWO BODIES TO CONCATENATE)

For `https://github.com/nutanix-core/iam-themis/pull/1603`. This PR carries two commits worth of work; the description reflects both.

### Section A — main body (entity + role v4)

Original Step 3 work (`7895a1e8f` "Add products field to IAM Entities and Roles"). OUTLINE — needs filling out from the prior session's notes:

- `storage.Object.ProductList`, `storage.Role.ProductList` storage fields
- ENG-932537 migration block (additive idempotent for `object.product_list` + `role.product_list`)
- SQLite test schema mirror
- `services/server/apihandler/util.go::ComputeAccessibleEntitiesList` extended to compute role.products as sorted-deduplicated union
- `services/server/apiutil/object_util.go::ToGetV4ObjectResponse` surfaces `Entity.products`
- `services/server/apiutil/role_util.go::ToGetV4RoleResponse` surfaces `Role.products`
- OData filter+select wiring for entities and roles
- Vendor TEMP HACK on `authz_model.go::Entity` and `Role` (canonical-style comment)
- Unit tests + tavern test `api_tests/roles_v4/test_role_product_list_collation.tavern.yaml`

### Section B — follow-up: v1 + /proxy entity surfacing

For the `ec87ae6a4` commit (originally `6a94fbf17` before the AP-rollback rebase). Append as a new "v1 + /proxy entity follow-up" subsection.

This commit closes the v1 GET/LIST gap for `Entity.products`, paired with the v4 entity wiring (7895a1e8f).

**Reconciliation discovery:** the v1 + /proxy write path was already wired on master:

- `LoadObjectRequest.Products []string` already in vendor with full swagger validation; `ReadOnly` enforcement bypassed because `generateValidatedObjectMap` calls `modelutil.Validate(...)` not `ContextValidate(...)`
- `storage.FromLoadObjectRequest` already copies `ob.Products -> newob.ProductList` (line 1277)
- `ObjectSupportedFilterFields` already contains `"products"`
- Column projection already wired in `ObjectAllColumns*`

The only actual gap was that `ClientObject` (the v1 response model) did not have a `Products` field, and the two v1 response builders (`ToGetObjectResponse`, `ToListObjectResponse`) were not assigning it. Both fixed.

**Production changes:**

- vendor TEMP HACK: `ClientObject.Products []string` in `vendor/github.com/nutanix-core/iam-utils/themisutil/generated/models/client_object.go`. Matches existing generated style; parallel iam-utils swagger PR (see Section 3 above) lands the canonical version, after which the next vendor bump produces a byte-identical regen.
- `storage/util.go::ToGetObjectResponse` assigns `o.ProductList -> ClientObject.Products`
- `storage/util.go::ToListObjectResponse` assigns `ob.ProductList -> ClientObject.Products`

**Tests:**

- 4 new tests in `storage/util_test.go` (11 sub-tests): `TestToGetObjectResponse_SurfacesProducts` (multi/single/empty/nil), `TestToListObjectResponse_SurfacesProductsPerElement`, `TestFromLoadObjectRequest_PreservesProducts` (multi/single/empty/nil), `TestToGetObjectResponse_ProductsRoundTripFromLoadRequest`
- New tavern E2E `api_tests/objects/test_object_products_v1_and_proxy.tavern.yaml`:
  - v1 direct load -> v1 GET single -> v1 LIST `$filter=products` + `$select` -> v4 cross-surface GET -> cleanup
  - /proxy-forwarded load -> v1 GET single read-back -> v4 cross-surface GET -> cleanup
  - All stages ASCII-only per the Python 2.7 codec fix.

**Notable decisions** (see [[08 - Decisions Log]] for full rationale):

- D21: NO runtime `ProductAllowedTypesMap` validation in iam-themis; matches v4 EntityConfig path; validation lives in iam-utils configload only
- D23: NO vendor edit on `LoadObjectRequest`; field already exists with full swagger validation, `ReadOnly` check bypassed because handler calls `Validate` not `ContextValidate`
- D24 (2026-05-29): AccessPolicies removed from this PR's scope. The Q-L3 follow-up commit `a411a8943` was dropped via `git rebase --onto 7895a1e8f`. The branch is now `7895a1e8f` → `ec87ae6a4` only.

Gates: gofmt clean on touched non-vendor + vendor files; go build clean; go vet shows only the pre-existing `proxy_role_test.go:297` unkeyed-fields lint; go test ./services/server/... all-green across 22 packages.

### Standard checklist (applies to both sections)

- [x] Unit tests added or updated.
- [x] API tests added or updated (tavern).
- [x] Performance tests considered (text[] OData filter; GIN-index deferral logged with re-eval triggers in `.notes/PRODUCTS_FIELD_CONTEXT.md` §12).
- [x] Manual testing performed.
- [x] Self-review completed.
- [x] Code adheres to project style guidelines.
- [x] Relevant comments added for clarity.
- [x] Code meets quality gate criteria.
- [ ] Vulnerability scans show no critical issues (confirm in CI).
- [ ] ReadMe and related docs updated as applicable. *(N/A — internal field addition.)*

---

## See also

- [[04 - PR Status]] for the live state of each PR
- [[09 - Open Questions & Followups]] for the must-do-before-merge list
- In-repo notes `iam-themis/.notes/PRODUCTS_FIELD_CONTEXT.md` carries the per-section originals if a paste needs more detail (§4.9 = bootstrap, §15 = AP, §16.8 = iam-utils ClientObject swagger, §16.9 = iam-themis section-C)
- Up: [[00 - Index]]
