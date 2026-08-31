---
tags: [eng-932537, decisions, log]
---

# Decisions Log

Numbered D1 through D23, grouped by the epoch in which they were made. Each row: decision, why, consequences.

## Epoch 1 — iam-bootstrap initial work (D1–D6)

These were made during the iam-bootstrap implementation session that produced PR #768.

### D1 — Reject Manish's `origin/ENG-932537` no-suffix branch on iam-bootstrap; fresh-start on `ENG-932537-products-field`

**Why:** Manish's branch (2026-05-21, `8a9738ca`) bundled unrelated cruft (Dockerfile changes, go.mod downgrades), implemented an over-engineered side-band map workaround instead of straight-line `ConfigObject → LoadObjectRequest` field copy, and inlined production logic into the test files rather than extracting pure helpers.

**Consequence:** Fresh branch from master. Same field/tag/comment conventions as our work in other repos. Manish's branch left alone (D7).

### D2 — Vendor TEMP HACK style: copy generated files byte-for-byte, mark with `// TEMP HACK ENG-932537 - ...`

**Why:** Need to develop against upstream model changes that haven't merged yet. Going strictly canonical (block on iam-utils + ntnx-api-iam PRs) adds calendar delay measured in days/weeks. Marker is grep-able for the eventual cleanup.

**Consequence:** Established the TEMP HACK pattern that the rest of the initiative copies. Later revised in D12 / D22 to drop the marker text in favor of byte-identical-to-canonical-regen style.

### D3 — Extract `toLoadObjectRequest` and `buildV4EntityConfig` as pure helpers for testability

**Why:** Manish's reference branch inlined this logic in tests; ugly. Pure helpers can be unit-tested without spinning up a fake bootstrap context.

**Consequence:** 5 unit tests added across 2 files; trivial to add edge cases.

### D4 — Skip ProductAllowedTypesMap runtime check in iam-bootstrap (validation already runs at config-load time in iam-utils)

**Why:** Validation is a config-load-time invariant in `configutil/isConfigValid`. Re-validating in the bootstrap path is redundant.

**Consequence:** Cleaner bootstrap code. Trust boundary lives in iam-utils, not iam-bootstrap. Same principle later applied to iam-themis runtime (D21).

### D5 — Set both v1 (`LoadObjectRequest.ProductList`) and v4 (`EntityConfig.ProductList`) bootstrap paths atomically in one PR

**Why:** Bootstrap chooses v1 or v4 dynamically based on cluster version flags. Fixing only one leg leaves the other broken in production for that version.

**Consequence:** PR #768 touches both paths. Tested with unit tests for both helpers.

### D6 — Use `--force-with-lease`, not `--force`, after rebases

**Why:** Standard safe variant; aborts if anyone else pushed in between.

**Consequence:** Recorded as the standing convention; mentioned in [[10 - Process Lessons]].

## Epoch 2 — Cross-repo housekeeping (D7–D8)

### D7 — Do NOT delete Manish's stale `origin/ENG-932537` (no-suffix) branches on ntnx-api-iam, iam-themis, iam-bootstrap

**Why:** Pre-delete audit showed all three branches carry open PRs by Manish Lokur (#907, #1594, #754). Deleting the branches would side-effect-close those PRs. User explicit direction: "don't touch anyone else's PRs".

**Consequence:** Cleanup rescoped to a human-coordination task. Zero git operations performed against those branches/PRs.

### D8 — Harden logging discipline at the cursor-rule layer (always-applied)

**Why:** User asked "how do I ensure you always update the logs every time you work". Tier 1 = cursor rule edit (flipping `alwaysApply: false` to `true`, adding a logging-discipline section). Tier 2 (user-level memory) and Tier 3 (`beforeStop` enforcement hook) deferred.

**Consequence:** `.cursor/rules/iam-products-field.mdc` now loads on every prompt in the workspace, not only when an iam-* source file is in context. §7 of the rule mandates updates to the in-repo `.notes/PRODUCTS_FIELD_CONTEXT.md` file on every sustained turn.

## Epoch 3 — Q-L3 AccessPolicy wiring (D9–D17) [REVERTED 2026-05-29]

> **REVERTED per D24.** AccessPolicies are out of scope. The decisions in this epoch were sound at the time but their consequences (storage column, handler chokepoint, vendor TEMP HACK on AuthorizationPolicy + Projection, derive-from-role plumbing, ~50+27 test-mock updates, AP tavern test) have all been rolled back. iam-themis branch dropped commit `a411a8943` via `git rebase --onto 7895a1e8f a411a8943`; ntnx-api-iam `658bbe23` was surgically amended to `01fb9033` (kept Entity + Role; reverted `access_policy.yaml` and `accessPolicyProductsDesc`). Decisions retained below for forensics — DO NOT use as guidance for current work.

These were made during the iam-themis AccessPolicy wiring session (PR #1603 follow-up commit `a411a8943`).

### D9 — Wire AP `products` end-to-end now (not defer to follow-up)

**Why:** Spec doc + ntnx-api-iam #914 vendor both carry the field; without themis wiring, the v4 OData filter and GET response on AP would 500/empty on every PC/NC cluster post-merge.

**Consequence:** AP wiring shipped on the same branch as the entity+role work, not as a separate PR.

### D10 — Derivation chokepoint = `validateAP` column-list expansion (not a new helper)

**Why:** `validateAP` is already called on every create and update path. Adding `"product_list"` to its existing `QueryRoles` column lists costs zero additional DB round-trips. A separate helper would have been a parallel DB read.

**Consequence:** Single edit covers both create and update. See [[06 - Handler Logic]].

### D11 — Stable-until-update semantics for AP.products (NO cascade on role-products change)

**Why:** Deliberate non-feature. Cascade would require fan-out events to all referencing APs on role change. Trade-off: freshness gap for (a) no fan-out, (b) deterministic etag stability, (c) cheap update path.

**Consequence:** Future revisit if NC ever needs eventual-consistency-with-cascade. Logged as P-track item in `iam-themis/.notes/...` §12.

### D12 — TEMP HACK markers on the AP vendor patch (initially; revised in D22)

**Why (at the time):** `AuthorizationPolicy` in the vendor file was still on pre-#914 generation. Markers would make the revendor cleanup grep-able later.

**Consequence:** Initially shipped with markers. Later revised in D22 after the vendor-diff-cleanup discovery (see [[11 - Issues Encountered & Fixes]] for the gofmt cascade incident).

### D13 — Vendor TEMP HACK targets `iam-server-codegen` path, NOT `ntnx-api-golang-sdk-internal`

**Why:** `ap_util.go` imports from `iam-server-codegen`; the SDK-internal vendor exists too but isn't on the AP code path. Verified by import-trace before editing.

**Consequence:** Single vendor path patched; no risk of patching the wrong vendor copy.

### D14 — Leave `GetRoleByUUID` calls in `v4_access_policy.go:321` and `:803` unchanged

**Why:** Those calls are used ONLY for `validateSuperAdminRoleBasedAp` validation, never for building the AP storage object. Widening their column list would be dead-cost.

**Consequence:** Bulk-replace regex also matched 10 of these mocks by accident; reverted individually (see [[11 - Issues Encountered & Fixes]]).

### D15 — Defer Q-TD2 (read-only enforcement: reject client-set products on AP create/update)

**Why:** `models.AccessPolicyRequest` has no Products field; clients literally cannot set products via the supported wire format. Enforcement would only be needed if a future API revision adds the field.

**Consequence:** No enforcement code today. If a future revision adds the field, add `if len(apRequest.ProductList) > 0 { return 400 "products is server-computed" }` after the existing `apRequest.Role == nil` check.

### D16 — /proxy AP partial-update path needs NO edit

**Why:** `proxy_access_policy.go::updater` only mutates Resource, AccessPolicyType, UpdatedTime, Opaquedata, ETag. ProductList is preserved because the updater pattern doesn't touch `old.ProductList` (which now scans-in correctly thanks to the column-list extension).

**Consequence:** Zero changes to /proxy AP code. Verified by reading the updater body.

### D17 — Test-mock bulk-update strategy: include the method name in the search key

**Why:** Pattern (a) `, []string{"iun", "uuid"}).Return` was unique to QueryRoles mocks for SOME files — but matched some GetRoleByUUID mocks that happen to use the same 2-element column list and the same suffix. Caught by post-hoc test failure; reverted individually.

**Consequence:** For future bulk-edits, the search key must include the method name when the column tuple is generic. Recorded in [[10 - Process Lessons]].

## Epoch 4 — §16 v1+/proxy entity widening (D18–D23)

Made during the entity v1/proxy widening session (PR #1603 follow-up commit `6a94fbf17`).

### D18 — Scope = entities-only Phase 1 (roles + APs at v1/proxy deferred)

**Why:** User's literal scope was "wire the entities to v1, v4, and /proxy". Glean's broader scope claim ("all three across all surfaces") cited the Global IAM design doc, but verification against the user-provided PDF showed that doc is ENG-915519 / Kronos and contains zero mention of `products`. No design doc backs widening roles+APs in this PR.

**Consequence:** Roles + APs at v1/proxy deferred to separate follow-up tickets.

### D19 — Vendor strategy = TEMP HACK in iam-themis vendor + parallel canonical iam-utils swagger PR drafted day-one

**Why:** Same logic as D2. Going strictly canonical adds days-to-weeks of calendar delay. Going strictly TEMP HACK without a paired upstream PR creates indefinite tech debt. Drafting both in parallel gives reviewers visibility into the cleanup path.

**Consequence:** [[12 - PR Description Drafts]] sub-section "iam-utils ClientObject swagger PR" carries the canonical PR text.

### D20 (REVISED) — JSON tag on `ClientObject.Products` and `LoadObjectRequest.Products` is `products` (uniform, not split)

**Why (original):** Hypothesized an inbound/outbound naming split (`productList` on request, `products` on response) based on `ConfigObject.ProductList` precedent.

**Why (revised after reconciliation discovery):** The existing `LoadObjectRequest.Products` field in iam-utils vendor is already tagged `json:"products,omitempty"`, NOT `productList`. The convention in iam-utils is uniform `products` across both directions. The earlier rationale was wrong about iam-utils precedent.

**Consequence:** Both vendor adds use `products` JSON tag. Matches the v4 wire format (`Entity.products`).

### D21 — NO runtime validation against `ProductAllowedTypesMap` in iam-themis

**Why:** Matches v4 EntityConfig load path behavior — neither surface invokes the allow-map at runtime. Validation lives entirely in `iam-utils/configutil/isConfigValid` at bootstrap config-load time (verified by grep: `ProductAllowedTypesMap` appears in zero iam-themis files). Adding runtime validation in §16 would diverge from v4 and could spuriously reject legitimate cross-deploy flows where the cluster's `productutil.GetProductType()` value lags the request.

**Consequence:** If runtime validation is later required, add it as a separate ENG ticket touching BOTH v1 (`generateValidatedObjectMap`) and v4 (`FromV4EntityRequest`) atomically.

### D22 — Vendor cleanup story = grep-by-field-name, NOT grep-by-marker (revises D12)

**Why:** During post-Q-L3 cleanup, discovered that adding TEMP HACK markers to the AP vendor patch caused gofmt to reformat the entire vendor file (2-space → tabs), bloating the diff by ~30,500 lines. The fix was to use the canonical generated-code comment style (no marker) so the vendor file looks generated and the diff is minimal.

**Consequence:** All §15 and §16 vendor adds use canonical style. Post-revendor cleanup is now: grep for the field NAME inside the parent struct, verify the regen produces the same line, accept verbatim.

### D23 — NO separate vendor edit on `load_object_request.go`

**Why:** The vendor file already has `Products []string` (line 46) with full swagger validation. The `validate.ReadOnly` enforcement chain (`ContextValidate`) is never called by `generateValidatedObjectMap` — which calls only `modelutil.Validate(…)` → `Validate(formats)` → MinItems/MaxItems. So clients can already send `products` via load-objects requests; no vendor edit needed.

**Consequence:** §16 production scope shrank to one vendor field add + two response-builder assignments. The "reconciliation discovery" pattern (audit before planning) is logged in [[10 - Process Lessons]].

## Epoch 5 — AccessPolicy scope rollback (D24)

### D24 — Rollback AccessPolicy across all 4 repos

**Decision:** AccessPolicies are out of scope for ENG-932537. Drop the iam-themis Q-L3 commit (`a411a8943`); surgically revert AP changes from ntnx-api-iam #914 (`658bbe23` → `01fb9033`); leave iam-utils #358 and iam-bootstrap #768 alone (no AP code present in either).

**Why:** User direction (2026-05-29 19:29 UTC): "Actually I dont need to touch access policies so there is no need to make those changes, only roles and entities need to have the changes. … The AP changes also have to be removed in ntnx, utils and bootstrap aswell." The original AP scope (Q-L3) was added in good faith based on the ticket interpretation at the time, but no design doc backs AP coverage; deferring it to a follow-up ticket if NC ever needs AP-level filtering is the simpler path.

**How:**

- iam-themis: `git rebase --onto 7895a1e8f a411a8943 ENG-932537-products-field` cleanly dropped `a411a8943` and replayed `6a94fbf17` as `ec87ae6a4` directly on `7895a1e8f`. Force-pushed with lease.
- ntnx-api-iam: `git checkout 658bbe23^ -- access_policy.yaml` reverted that file completely; `StrReplace` removed the `accessPolicyProductsDesc` block from `iamDefsDescriptions.yaml`; `git commit-tree` plumbing rebuilt the commit as `01fb9033` preserving original author/date and updating the message to drop "AccessPolicy". Force-pushed with lease.
- iam-utils #358: untouched. Pre-rollback grep for `AccessPolicy` in commit `696d458` returned nothing — title's claim ("LoadObjectRequest and RoleRequest") was accurate.
- iam-bootstrap #768: untouched. Pre-rollback grep for `AccessPolicy` in commit `c87e939e` returned nothing — entity-only seeding paths.

**Consequences:**

- All AP-related decisions D9–D17 are now obsolete (decisions valid at the time, but their consequences are rolled back).
- Cursor rule §2 third bullet: rewritten to flag APs as out-of-scope with reflog pointers.
- Cursor rule §3: migration scope narrowed to `object` + `role` tables only.
- Cursor rule §6: reference-files list trimmed (no more `AuthorizationPolicy` callout).
- iam-themis #1603 PR description draft (Section B in [[12 - PR Description Drafts]]): removed.
- API Surface Matrix: AP rows changed from ✅/⏭ to ❌ (removed).
- Storage & Migration column-list constants: AP set untouched and explicitly noted as out-of-scope.
- Reflog forensics preserved on both repos for if AP scope is ever reopened (`a411a8943` on iam-themis; `658bbe23` on ntnx-api-iam).

**Resolves:** Scope question that has been latent throughout the initiative.

**Resolves comments:** N/A (no public review comments yet).

## Epoch 6 — ASCII regression discovery (D25)

### D25 — Scrub a second pass of em-dashes in `test_role_product_list_collation.tavern.yaml`

**Decision:** Apply a single-file `perl -i -CSD -pe 's/\x{2014}/-/g'` scrub, commit + push as a new fast-forward commit on top of the post-rollback iam-themis tip.

**Why:** A "did you run all the tests" verification pass turned up 7 em-dashes (U+2014) still in our role tavern file — 5 of them in stage `name:` fields. The §15 fix was incomplete despite its changelog claim of "verified clean". Python 2.7's tavern collector formats every stage name and crashes with `UnicodeEncodeError` on any non-ASCII byte; so 5 stage names × U+2014 = a guaranteed CI explosion before any actual test runs. That's exactly the failure mode §15 set out to fix, and it has been on origin since `7895a1e8f` was pushed in session 2.

**How:**
- `perl -i -CSD -pe 's/\x{2014}/-/g' api_tests/roles_v4/test_role_product_list_collation.tavern.yaml` (single file; comparable file `api_tests/iamv4/role_memberships/test_project_ap_to_role_membership_migration.tavern.yaml` had 5 em-dashes too but all in YAML comments and a different ENG ticket — left alone)
- Verified parse via `ruby -ryaml YAML.load_stream`
- Verified zero non-ASCII via re-scan
- Pure YAML change so no Go gates needed re-running (the §17 final gate evidence still applies byte-identically)
- Committed via `git commit-tree` plumbing — new SHA `7aa4c7c77`, subject `ENG-932537: scrub non-ASCII em-dashes from role tavern stage names`, no `Co-authored-by` trailer, correct author
- Pushed as ordinary fast-forward `ec87ae6a4..7aa4c7c77` (lease satisfied trivially because we added a commit, didn't rewrite history)

**Consequences:**

- New iam-themis origin tip: `7aa4c7c77` (was `ec87ae6a4`)
- Cross-repo IAMv2 Automation Tests will re-fire on the new tip; if they go green now, the latent ASCII bug was the actual culprit for the red status that's been blocking #1603 (not the cross-repo coordination hypothesis, or in addition to it)
- Verification recipe formalized in [[11 - Issues Encountered & Fixes]] G1: pair `perl -CSD` scan with `wc -l` sanity-check; re-scan after every edit; never trust an "(empty)" output as proof of clean without confirming the scan covered the whole file

**Resolves:** Latent ASCII regression that had been on origin since session 2.

**Resolves comments:** N/A (no public review comments yet).

## See also

- [[09 - Open Questions & Followups]] for items deferred by these decisions
- [[10 - Process Lessons]] for the abstract patterns derived from these decisions
- [[11 - Issues Encountered & Fixes]] for the concrete incidents that some decisions reacted to (D14, D17, D22)
- Up: [[00 - Index]]
