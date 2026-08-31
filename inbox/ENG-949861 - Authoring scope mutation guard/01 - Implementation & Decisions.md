---
tags: [eng-949861, implementation, decisions, verification]
updated: 2026-07-09
---

# Implementation & Decisions

## Requirement (Jira was unreachable — from user guidance 2026-07-09)
`jira.nutanix.com/browse/ENG-949861` returned 503/timeout the whole session. Requirement taken from the user + the Global IAM design context:

- Block a mutation (**PUT / DELETE**) when **both**: (1) request runs on a **PC** cluster, and (2) the existing entity's `authoringScope` is **NC**.
- Scope: **Roles and Access Control Policies (ACPs / v4 Authorization Policies)** — the two types that carry `authoringScope`.
- Enforce at the **API handler layer** (not persistence), so the internal Lattice `ApplyChange` sync — which must keep updating/deleting NC-authored entities on PC to stay in sync — is unaffected.
- Error: 403 Forbidden, message: `"This entity is managed by Nutanix Central and cannot be modified or deleted locally on Prism Central."`

## Key finding — the described block ALREADY exists via `AllowOperation` (surfaced to user)
`services/utils/lattice_utils.go::AllowOperation(isGlobal, caller)` returns **false** on PC for `isGlobal && caller==CallerAPI`. All six create/update/delete handlers (roles + ACPs) already call it and reject with `"Global ... are only allowed for NC products"` (HTTP 400) before the lattice-sync branch.

Chain that makes it airtight: `authoringScope=="NC"` ⟺ the entity is global (`ComputeAuthoringScope` only stamps `"NC"` when `isGlobal` on NC; global↔local conversion is rejected; global entities can't be created on PC). So **on PC, every NC-authored entity's update/delete is already blocked** — a pure new "block" would be dead code today.

**Resolution:** implement the `authoringScope`-keyed guard **before** `AllowOperation` so it is *not* redundant: for the NC-on-PC case it produces the clearer, spec'd **403 + "managed by Nutanix Central"** response instead of the misleading 400, and it is **durable** — if `AllowOperation`'s blanket PC block is later relaxed for Global IAM GA, this guard still keeps NC-authored entities read-only on PC. Other cases fall through to the existing `AllowOperation` unchanged.

## Correction carried forward
User described `authoringScope` as "an array of strings; check contains NC". Per ENG-915519 (scalar flip 2026-06-01) it is a **scalar** enum/string in storage, models, and Postgres. So "contains NC" = `== "NC"` and "on PC" = `productutil.IsPCProduct()`. Semantically identical to the intent.

## Decisions
- **D1 — Guard placed before `AllowOperation`, keyed on stored `authoringScope`.** Non-redundant (better message/status) and future-proof. See finding above.
- **D2 — Reuse `ModificationNotAuthorized` + HTTP 403.** Matches the authorization-semantics error code already used in `enforcement/util.go`; 403 per user preference (existing global block used 400, which reads as a product limitation rather than an ownership rule).
- **D3 — Single shared helper `util.IsAuthoringScopeMutationBlocked(authoringScope)`** next to `ComputeAuthoringScope`, rather than inlining the check in four handlers. `= productutil.IsPCProduct() && authoringScope == productutil.NcProduct`.
- **D4 — Directional (NC→PC only), not symmetric.** Matches the Kronos replication direction (NC authors, PC consumes). A PC-authored global entity is not blocked (unreachable today, but the guard must not block it).
- **D5 — v1/proxy paths not covered (v4 only).** Consistent with ENG-910350 D5; global authoring is a v4 concern. Flag as follow-up if v1 global mutation turns out reachable.
- **D6 — New branch/worktree off ENG-915519**, since the `authoringScope` field only exists there (not on `iam-themis` main branch).
- **D7 (REVERSED 2026-07-09 session 2) — AP-update handler test ADDED.** Originally omitted as "fragile". Reversed after finding an existing proven template, `TestUpdateAuthorizationPolicyById_IsGlobal_OperationNotAllowed` (access_policy_test.go:4782), which already navigates the full update validation chain on PC and reaches `AllowOperation` (line 403). Mirroring it with `AuthoringScope: NC` on the existingAcp trips the guard at line 389 first — not fragile, since the template proves the chain is reachable with those exact mocks (`GetRoleByUUID`, dedupe `GetAccessPolicy`, `GetAccessPolicyByUUID` w/ `AccessPolicyAllColumnsForProxy`). All four guarded paths now have handler tests.
- **D8 — Tavern (integration) test NOT added; feasibility documented.** The natural home is `api_tests/zz_global_iam/test_global_role_acp_crud.tavern.yaml`. Problem: the guard only fires for an **NC-authored entity present on a PC cluster**. On the single-cluster tavern rig, `themis_helper.py::setup_global_iam_product` flips product **PC→NC only** and restarts the pod; there is **no reverse (NC→PC) helper**. To exercise the guard you must: create the global (NC-stamped) entity under NC mode → flip product **back to PC** (new Python helper + pod restart) → PUT/DELETE → assert 403 + message → then flip back to NC to clean up (the guard blocks the normal PC delete, so cleanup ordering must change). That is non-trivial test-infra work (new helper, 2 extra pod restarts, reworked teardown) that **cannot be executed/verified in this sandbox** — tavern needs the full k3d IAM deployment (certs, lattice CG, AD connector). Deferred pending a decision to invest in the helper + a k3d run; Go handler tests cover the guard behavior in the meantime.

## Files touched (branch `ENG-949861-authoring-scope-mutation-guard`)
| File | Kind | Why |
|---|---|---|
| `services/server/util/authoring_scope.go` | edit | Add `IsAuthoringScopeMutationBlocked(authoringScope string) bool`. |
| `services/server/util/constants.go` | edit | Add `NCAuthoredMutationBlockedErrorMsg` constant. |
| `services/server/apihandler/v4_roles.go` | edit | Guard in `DeleteRoleById` (after fetch, before `AllowOperation`) and `UpdateRoleById` (after by-uuid fetch, before conversion check). |
| `services/server/apihandler/v4_access_policy.go` | edit | Guard in `UpdateAuthorizationPolicyById` (after existingAcp fetch) and `DeleteAuthorizationPolicyById` (after existingAcp fetch). |
| `services/server/util/authoring_scope_test.go` | edit | `TestIsAuthoringScopeMutationBlocked` (5 cases). |
| `services/server/apihandler/authoring_scope_mutation_guard_test.go` | new | Handler tests for all four guarded paths — role delete/update + AP delete/update on PC with `authoringScope=NC` → 403 + message. |

## Verification (base `ENG-915519-authoring-scope-handler` @ `a41faa5cf`)
- `gofmt -l` on all touched files: clean.
- `go build ./services/...`: rc=0.
- `go vet ./services/server/util/ ./services/server/apihandler/`: only the **pre-existing** `proxy_role_test.go:297` unkeyed-struct-literal warning (untouched; documented in ENG-915519/910350 notes).
- `go test ./services/server/util/`: ok. `go test ./services/server/apihandler/`: ok (6.9s). No regressions — existing PC global-block tests still pass because their mocks leave `AuthoringScope` empty, so the new guard doesn't trip.
- **Session 2:** re-ran after adding the AP-update test. `go build ./services/...` rc=0; `go vet` same lone pre-existing warning; `go test -run NCAuthoredOnPC -v` → all **4** guard handler tests PASS; full `apihandler` + `util` suites green.

## Reproduce
```
cd .wt-themis-949861   # worktree off ENG-915519-authoring-scope-handler @ a41faa5cf
go build ./services/...
go test ./services/server/util/ -run TestIsAuthoringScopeMutationBlocked -v
go test ./services/server/apihandler/ -run NCAuthoredOnPC -v   # all 4 guarded paths
```

## Open items
- Not committed / not pushed.
- All four guarded paths now have handler tests (D7 reversed).
- Tavern/integration coverage deferred — requires NC→PC product-flip helper + k3d run (D8).
- v1/proxy coverage deferred (D5).
- Confirm error code/status (403 + `ModificationNotAuthorized`) against ENG-949861 acceptance criteria once Jira is reachable.

## Changelog
- **2026-07-09 (agent session 1):** Read the vault (ENG-915519 / ENG-910350 / ENG-932537) for conventions. Verified `authoringScope` is scalar and that `AllowOperation` already blocks global-on-PC. Added `IsAuthoringScopeMutationBlocked` + message constant; wired the guard (before `AllowOperation`, keyed on stored scope, 403 + clear message) into the four v4 role/AP update+delete handlers. Added helper + 3 handler tests. All local build/vet/test gates green. Not committed.
- **2026-07-09 (agent session 2):** In response to "did you test everything + add tavern tests": closed the AP-update handler-test gap by mirroring the proven `TestUpdateAuthorizationPolicyById_IsGlobal_OperationNotAllowed` template (D7 reversed) — all 4 guarded paths now covered and green. Investigated tavern feasibility: the `zz_global_iam` suite is the right home but the harness only flips PC→NC (no reverse helper), so exercising an NC-authored-on-PC mutation needs new test-infra + a k3d run that can't be validated in-sandbox (D8). No tavern test added; documented the path. Still not committed.
