---
tags: [eng-915519, pr-status]
status: live
updated: 2026-06-01
---

# PR Status — live state of all three PRs

> **2026-06-01 update:** all three PRs were pushed today carrying the array→scalar-enum conversion (see [[10 - Decisions Log]]). Each push is a clean append on top of the prior tip — no force-pushes, no rebases needed.

## PR #910 ntnx-api-iam

- **URL:** https://github.com/nutanix-core/ntnx-api-iam/pull/910
- **Branch:** `add-authoring-scope` → `main`
- **HEAD:** `ad2b392a` ENG-915519: Convert authoringScope from array to scalar enum
- **Prior tip:** `4cffeeb8` (Extract AuthoringScope enum into dedicated authoring_scope.yaml)
- **CI:** pending after the new push — re-check in a few minutes
- **Files in latest commit:** `access_policy.yaml`, `roles.yaml`, `authoring_scope.yaml`, `iamDefsDescriptions.yaml` (5 insertions, 16 deletions)
- **Pending small work** (still actionable, not in scope of the scalar flip):
  - #6: add `authoringScope` to `x-filterable-properties` on ACP
  - #7: same for Role
- **Pending replies on GitHub:**
  - #1 (Aditya, unquote authoringScopeDesc) — resolve as "fixed in earlier commit"
  - #2 / #4 / #5 (array vs string) — reply citing the scalar conversion in `ad2b392a` and asking for resolution

## PR #357 iam-utils

- **URL:** https://github.com/nutanix-core/iam-utils/pull/357
- **Branch:** `ENG-915519-add-authoring-scope` → `master`
- **HEAD:** `573ca20` ENG-915519: Convert authoringScope from array to scalar string enum
- **Prior tip:** `f9885b3` (Add authoringScope to ACP and Role schemas)
- **CI:** pending after the new push — re-check in a few minutes
- **Files in latest commit:** `themisutil/themis.yaml`, `themisutil/generated/models/role_request.go`, `themisutil/generated/models/access_policy_request.go` (44 insertions, 63 deletions)
- **Hand-edit caveat:** the generated Go models were hand-edited for the AuthoringScope-specific portions only. Full go-swagger regen still pending Manish on the version pin — when that lands, the regen sweep should match these hand-edits (shape mirrors `AccessPolicyType` / `OperationSchemaChangeImpact` already in the file).
- **Hard blocker (unchanged):** `go-swagger` version pin from Manish (#9 omitempty, #11 / #12 missing generated header). See [[09 - Open Questions & Blockers]].
- **Pending replies on GitHub:**
  - #10 (array shape) — reply citing the scalar conversion in `573ca20`
  - #13 (Aditya, "let me know if these were not present in the generated file") — reply confirming the header WAS in upstream output, lost by the wrong go-swagger version

## PR #1601 iam-themis

- **URL:** https://github.com/nutanix-core/iam-themis/pull/1601
- **Branch:** `ENG-915519-authoring-scope-handler` → `master`
- **HEAD:** `b17451c59` ENG-915519: Convert authoringScope from array to scalar string
- **Prior tip:** `e7ad49866` (Clarify migration comment, drop Xi reference, gofmt test file — the rebased tip from the 2026-05-28 session)
- **CI:** pending after the new push — re-check in a few minutes; SonarQube ❌ (empty-metrics setup issue, unchanged, see [[09 - Open Questions & Blockers]])
- **Files in latest commit:** 9 files — `storage/storage.go`, `storage/util.go`, `storage/sql/migrate.go`, `storage/sql/role.go`, `storage/sql/access_policy.go`, `storage/sql/client.go`, `storage/authoring_scope_test.go`, `storage/sql/authoring_scope_sql_test.go`, `apihandler/authoring_scope_test.go` (101 insertions, 110 deletions)
- **Vendor note:** iam-themis's vendored copy of iam-utils does NOT contain the AuthoringScope field at all (predates PR #357), so this conversion in iam-themis is self-contained — nothing in iam-themis reads the AuthoringScope field from request models. When iam-themis re-vendors after PR #357 merges, the only audit needed is to confirm no new code referenced `models.RoleRequest.AuthoringScope` / `models.AccessPolicyRequest.AuthoringScope` as `[]string` in the meantime.
- **Pending replies on GitHub:**
  - T5 / T6 / T8 / T10 (array pushback) — resolve as fixed in `b17451c59`
  - T7, T9, T11 — already resolved by `e7ad49866` (prior session)
  - T1 / T3 / T13 (bot AI noise) — resolve as "noted"
- **Held out (unchanged):** T4 (immutability doc comment), T2 (invalid product type test), T12 (SonarQube investigation) — see [[09 - Open Questions & Blockers]]

## Merge ordering

Strict sequential (unchanged):

1. **PR #910** must merge first (public schema)
2. **PR #357** second (revendors / picks up new schema)
3. **PR #1601** third (revendors both upstream repos)

After PR #1601 revendors, a **follow-up commit** is needed to remove the `// AuthoringScope intentionally not copied until upstream model revendor lands` comments in `v4_access_policy.go` / `v4_roles.go` and wire up the actual API → storage copy (defensive re-stamp pattern). The conversion to scalar makes that wiring slightly simpler — just a string copy with no slice-to-slice plumbing.

## See also

- [[08 - PR Reviews Rollup]] for the per-comment table
- [[09 - Open Questions & Blockers]] for what's gating progress
- [[10 - Decisions Log]] for what's been settled
- Up: [[00 - Index]]
