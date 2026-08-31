---
tags: [eng-932537, open, blockers, followups]
status: live
updated: 2026-05-29
---

# Open Questions & Followups

What's left, what's blocking, what was deliberately deferred, and what was rolled back. Prioritized.

## Resolved (most recent first)

### ✅ AccessPolicy scope — REMOVED 2026-05-29 (D24)

User scope decision: APs are not in scope for ENG-932537. iam-themis branch dropped `a411a8943` via rebase; ntnx-api-iam `658bbe23` surgically amended to `01fb9033`. iam-utils #358 and iam-bootstrap #768 had no AP code and were untouched. Reflog forensics preserved on both repos. Cursor rule + in-repo notes (§17 / D24) + this vault all updated. Was previously a top-of-list decision under "Q-L3" / D9–D17 — those decisions retained in [[08 - Decisions Log]] for forensics but flagged as REVERTED.

## Hard blockers (must do before merge)

### 1. CI failures on iam-themis #1603 and iam-bootstrap #768

- Both PRs show Cycode ✅ + CircleCI build ✅ but **IAMv2 Automation Tests ❌**.
- iam-themis CI will rerun on the new `ec87ae6a4` after the force-push; same hypothesis applies.
- **Working hypothesis (from §14 PR survey):** cross-repo coordination. The test rig deploys one repo at a time against trunk peers, so:
  - themis alone with the new `product_list` column against trunk-bootstrap-without-population fails
  - bootstrap alone against trunk-themis-without-column fails
  - Both passing simultaneously requires both branches to be in flight
- **Testable when:** all 4 PRs are simultaneously merged into trunk (or at least into the rig's staging trunk), at which point the multi-leg deploy mismatch resolves.
- **Diagnostic step:** look at the actual test failure output from a recent IAMv2 Automation Tests run (via `gh pr checks` or the CircleCI UI) and confirm the failure mode matches the hypothesis (e.g., "column product_list does not exist" or "unknown field products").

### 2. Backfill PR descriptions on all 4 PRs

All 4 PR bodies are still GitHub template placeholders. The canonical bodies live in [[12 - PR Description Drafts]]; copy-paste into:

- ntnx-api-iam #914 — needs to be written from the outline (Entity + Role only, AP removed per D24)
- iam-utils #358 — needs to be written from the outline
- iam-themis #1603 — two sub-sections to paste (main entity+role v4, v1+/proxy entity follow-up); the former AccessPolicy section is removed
- iam-bootstrap #768 — fully drafted; copy-paste ready

## Should do before merge (small)

### 3. Open the parallel iam-utils swagger PR for `ClientObject.products`

- Closes the iam-themis #1603 vendor TEMP HACK on `client_object.go`
- Full schema snippet + PR body draft in [[12 - PR Description Drafts]] sub-section "iam-utils ClientObject swagger PR"
- Branch suggestion: `add-products-to-client-object`
- Once merged + revendored, the iam-themis vendor add becomes byte-identical no-op overwrite

## Post-merge cleanup (gated on upstream merges)

### 4. Real revendor in iam-themis (after #914 + #358 + new ClientObject swagger PR merge)

Grep cleanup is by field NAME, not by TEMP HACK marker (D22):

- `vendor/.../authz_model.go` — search for `ProductList []string` inside `type Entity struct`, `type Role struct`, `type EntityProjection struct`, `type RoleProjection struct`. (NOT `AuthorizationPolicy*` — those entries were never re-added after D24 cleanup.)
- `vendor/.../client_object.go` — search for `Products []string` inside `type ClientObject struct`

If the regen produces byte-identical lines (which it should), the "cleanup" is rerunning `make revendor` and accepting the diff.

### 5. Real revendor in iam-bootstrap (same condition)

`EntityConfig.ProductList` TEMP HACK in iam-bootstrap's vendor cleans up the same way.

## Open / lower priority

### 6. Stale `origin/ENG-932537` (no-suffix) branches (human coordination)

Per D7, do NOT touch — they carry Manish's open PRs (#907, #1594, #754). Tracked as a human-coordination item; agent will not act on them.

### 7. Pre-existing tech-debt items (defer to follow-up ENG tickets)

#### Q-TD1 — `UpdateOperationList` drift

Server allows updating `operationList` on a role via certain paths without invoking the full create-time validation chain. Means a role's `accessibleEntities` and `products` can fall out of sync if the operations change via the drift path.

**Fix sketch:** Add the `ComputeAccessibleEntitiesList` recompute hook to every path that touches `operationList`. Small (likely 1-3 sites).

#### Q-TD3 — Lattice multi-cluster validation

Assert NC ↔ PC peer replication carries `products` correctly. Needs a multi-cluster test rig we don't have today. Single-cluster tavern tests in this initiative validate in-cluster contract only.

**Defer reason:** rig doesn't exist; building it is its own project.

#### P10 — swagger `readOnly: true` brittleness on `LoadObjectRequest.Products`

If any future iam-themis caller switches from `modelutil.Validate(...)` → `ContextValidate(...)` on the load path, the swagger `readOnly: true` marker will start rejecting non-empty values on requests. Fix would be either:

- Strip the `readOnly` marker upstream in the iam-utils swagger
- Pin `Validate`-only in the iam-themis handler (current state)

**Defer reason:** current state works; changing iam-utils canonical schema needs cross-team alignment.

#### Drive-by: `iam-utils/configutil/ConfigACP.ProductList` is unused

After D24, `ConfigACP.ProductList` exists in iam-utils #358 but is referenced nowhere in iam-themis or iam-bootstrap. Removing it requires a follow-up iam-utils PR with no current beneficiary. Harmless to leave; a future tidy-up PR can drop it cleanly.

### 8. Roles at v1 + /proxy widening (deferred per D18)

Explicitly out of scope for this initiative. No design doc backs widening. Each would be a separate small ENG ticket if NC ever needs them.

### 9. AccessPolicies at any surface (REMOVED per D24)

If reopened, see reflog SHAs in §17 of `iam-themis/.notes/PRODUCTS_FIELD_CONTEXT.md`.

## Aside (pre-existing, NOT introduced by this work)

These show up in test gates but are pre-existing on bare HEAD; verified by stash-pop-and-rerun:

- `services/server/apihandler/proxy_role_test.go:297:32` go vet warning: `storage.RoleProxyResponse struct literal uses unkeyed fields`. Drive-by cleanup, not in-scope for ENG-932537.
- `TestTenantConfigLoad_ProxySeed_Failure` fails when run in isolation (`go test -run "^TestTenantConfigLoad_ProxySeed_Failure$"`) but passes inside the full apihandler sweep. Pre-existing inter-test state dependency. Not in scope.

## See also

- [[04 - PR Status]] for the live PR state these followups attach to
- [[12 - PR Description Drafts]] for the bodies that need backfill
- [[08 - Decisions Log]] for D18, D21, D22, D24 (which produced several of these followups)
- Up: [[00 - Index]]
