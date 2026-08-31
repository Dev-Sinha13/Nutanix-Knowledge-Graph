---
tags: [eng-932537, pr-status]
status: live
updated: 2026-05-29
---

# PR Status — live state of all 4 PRs

## PR #914 ntnx-api-iam

- **URL:** https://github.com/nutanix-core/ntnx-api-iam/pull/914
- **Branch:** `ENG-932537-products-field` → `main`
- **HEAD:** `01fb9033` ENG-932537: add products field to v4 authz Entity and Role
- **Scope:** v4 schema for `Entity.products` and `Role.products` + EDM filterability bindings. **AccessPolicy schema removed 2026-05-29 (D24)** — see [[08 - Decisions Log]].
- **CI:** Cycode ✅ | CircleCI build ✅ (will rerun on the force-pushed `01fb9033`)
- **PR body:** GitHub placeholder; draft pending in [[12 - PR Description Drafts]]
- **Reviews:** 0 human; nutanix-code-review bot placeholder only
- **Notes:** Parallel `origin/ENG-932537` (no-suffix) branch by Manish Lokur (#907) exists; D7 directs us to NOT touch it (carries Manish's open PR).

## PR #358 iam-utils

- **URL:** https://github.com/nutanix-core/iam-utils/pull/358
- **Branch:** `ENG-932537-products-field` → `master`
- **Scope:** `ConfigObject.ProductList`, `ConfigRole.ProductList`, `ConfigACP.ProductList`, `ProductAllowedTypesMap`, `isConfigValid` validation. (Note: `ConfigACP.ProductList` is unused in iam-themis runtime per D24, but was not removed — leaving it is harmless. Tracked as low-priority drive-by in [[09 - Open Questions & Followups]].)
- **CI:** Cycode ✅ | CircleCI build ✅
- **PR body:** GitHub placeholder; draft pending in [[12 - PR Description Drafts]]
- **Reviews:** 0 human
- **Followup PR pending:** parallel iam-utils PR to add `Products []string` to the `ClientObject` swagger schema (`themis.yaml`). Draft text in [[12 - PR Description Drafts]] sub-section "iam-utils ClientObject swagger PR". Closes the iam-themis #1603 vendor TEMP HACK for `client_object.go`.

## PR #1603 iam-themis

- **URL:** https://github.com/nutanix-core/iam-themis/pull/1603
- **Branch:** `ENG-932537-products-field` → `master`
- **Scope:** Storage + handlers + tests + tavern across v4 entities/roles and v1+/proxy entities. **AccessPolicy wiring removed 2026-05-29 (D24)** — see [[08 - Decisions Log]].
- **Origin tip:** `7aa4c7c77` (force-pushed 2026-05-29; ordinary fast-forward `ec87ae6a4..7aa4c7c77`)
  - `7895a1e8f` (origin/master..) — ENG-932537: Add products field to IAM Entities and Roles
  - `ec87ae6a4` — ENG-932537: surface products on v1 entity GET/LIST (replayed from `6a94fbf17` after the AP-rollback rebase)
  - `7aa4c7c77` — ENG-932537: scrub non-ASCII em-dashes from role tavern stage names (D25, §17.1; ASCII regression discovered during local-stack discovery; fixes a §15 incomplete-scrub miss)
- **Removed via rebase:** `a411a8943` (Q-L3 AP wiring) — reachable via reflog if AP scope is ever reopened
- **CI** (on the new `7aa4c7c77`): will rerun after force-push. Prior CI: Cycode ✅ | CircleCI build ✅ | **IAMv2 Automation Tests ❌**
- **Hard blocker:** IAMv2 Automation Tests failing. Hypothesis A: cross-repo coordination (test rig deploys one repo at a time against trunk peers). Hypothesis B (newly considered after D25): the `7895a1e8f` push has carried em-dashes in role tavern stage names since session 2; Python 2.7 tavern collector would `UnicodeEncodeError` before any test runs. The `7aa4c7c77` push removes Hypothesis B; if IAMv2 turns green now, B was the actual culprit. Re-testable when CI reruns on the new tip.
- **PR body:** GitHub placeholder; draft sections live in [[12 - PR Description Drafts]]:
  - main body (entity + role v4) — from prior session
  - v1/proxy entity section — from [[08 - Decisions Log]] D18-D23
  - (former AccessPolicy section removed per D24)
- **Reviews:** 0 human

## PR #768 iam-bootstrap

- **URL:** https://github.com/nutanix-core/iam-bootstrap/pull/768
- **Branch:** `ENG-932537-products-field` → `master` (chose fresh branch over Manish's `origin/ENG-932537` per D1)
- **HEAD:** `c87e939e` — Add products field seeding to v1 + v4 bootstrap paths
- **CI:** Cycode ✅ | CircleCI build ✅ | **IAMv2 Automation Tests ❌** (same cross-repo coordination hypothesis as #1603)
- **PR body:** Drafted; lives in [[12 - PR Description Drafts]]; **not yet copy-pasted to GitHub**
- **Reviews:** 0 human
- **Notes:** Parallel `origin/ENG-932537` branch by Manish (#754) exists; D7 directs us to NOT touch it. No AP code in this PR (verified by audit during the D24 rollback).

## Merge ordering

Strict-ish sequence (bottom-up by dependency):

1. **PR #914** — v4 schema lands (regenerates DTOs that iam-themis vendors)
2. **PR #358** — iam-utils types + validation land (imported by both iam-themis and iam-bootstrap)
3. **PR #1603** — iam-themis revendors and removes TEMP HACK markers
4. **PR #768** — iam-bootstrap revendors and removes TEMP HACK markers
5. **New iam-utils ClientObject swagger PR** — lands the canonical `Products` field on `ClientObject`; iam-themis revendors a second time and the §16 TEMP HACK becomes byte-identical to regen

In practice all 4 will likely land in a coordinated batch; the order matters mostly for whether the IAMv2 Automation Tests pass in each isolated leg.

## See also

- [[09 - Open Questions & Followups]] for what's blocking each PR's merge
- [[12 - PR Description Drafts]] for the bodies that still need backfill
- [[08 - Decisions Log]] D7 for the stale-branch rationale
- Up: [[00 - Index]]
