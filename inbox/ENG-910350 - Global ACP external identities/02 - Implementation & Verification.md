# Implementation & Verification

## Files touched (branch `ENG-910350-global-acp-external-identity`, worktree `.wt-themis-global-acp`)

| File | Kind | Why |
|---|---|---|
| `services/server/apihandler/global_acp_identity_validator.go` | new | `validateGlobalACPExternalIdentities` + allow-set maps + type-extraction/name helpers. |
| `services/server/apihandler/global_acp_identity_validator_test.go` | new | Table tests for the allow/deny matrix + httptest integration tests (external allowed, local/SA/unknown rejected, 404, wildcard-skipped, mixed-list). |
| `services/server/apihandler/v4_access_policy.go` | edit | Call the validator in the global create branch (after `CheckIfGlobalChangeAllowed`, before enrich/propose) and the global update branch (after `processUpdateAccessPolicyRequest`, before building the lattice change). |

## The validator
`validateGlobalACPExternalIdentities(ctx, apiContext, userList, userGroupList) iamerrors.ApiError`:
- Dedupes each list; skips `""` and `"*"` (dynamic match-any filters).
- For each user UUID: `apiContext.IAMAuthn.Users.ServiceClient.GetUserById(ctx, &usersreq.GetUserByIdRequest{ExtId:&id})`; 404 → `RequestValidationError` "does not exist"; other error → `mapIAMAuthnSDKError` (InternalError, fail-closed). Reads `authn.User.UserType`; rejects unless in `externallyManagedUserTypes` = {SAML, LDAP, EXTERNAL, PROPAGATED}.
- For each group UUID: `GetUserGroupById`; reads `authn.UserGroup.GroupType`; rejects unless in `externallyManagedGroupTypes` = {SAML, LDAP, PROPAGATED}.
- `nil`/UNKNOWN/REDACTED types fail closed (rejected).
- `authn` = `ntnx-api-golang-sdk-internal/iam-go-client/v17/models/iam/v4/authn` (the SDK client model, **not** iam-server-codegen). Mirrors `v4_role_membership_validator.go` (reuses its `mapIAMAuthnSDKError`).

## Call sites (`v4_access_policy.go`)
- **Create** (`ServeHTTP` POST, global branch): after the `CheckIfGlobalChangeAllowed` gate, validates `apStorageObj.UserList` / `UserGroupList`. On error: mark audit `HasError`, `v4ErrorResponse(..., http.StatusBadRequest)`.
- **Update** (PUT, global branch): after `processUpdateAccessPolicyRequest`, validates `processed.UserList` / `UserGroupList`. Same error handling. Global↔local conversion is already rejected upstream, so effective `isGlobal` == request == existing.

Both sites are already inside `if isGlobal { ... }` **after** `CheckIfGlobalChangeAllowed`, so the guardrail runs only on NC with global sync enabled — no separate feature flag added.

## Bootstrap audit (result: no change needed)
- iam-bootstrap seeds ACPs via `models.LoadAccessPolicyRequest` (`configload.go::loadAcps`), which **never sets `IsGlobal`** → all seeded ACPs are local.
- That seeding path does not traverse the guarded v4 create/update handler anyway.
- Therefore no default global ACP references a local/SA identity, and the new guardrail cannot affect bootstrap.

## Verification (base `origin/master` @ `734c90222`)
- `gofmt -l` on both new files + `v4_access_policy.go`: clean.
- `go build ./services/...`: rc=0.
- `go vet ./services/server/apihandler/`: only a **pre-existing** unkeyed-struct-literal warning in `proxy_role_test.go` (untouched).
- `go test ./services/server/apihandler/ -run 'TestValidateGlobalACPExternalIdentities|TestIsExternallyManaged'`: ok.
- `go test ./services/server/apihandler/` (full package): ok (6.9s).

## Changelog
- **2026-07-09 (agent session 1):** Set up worktree/branch off master. Verified all definitions on master (SDK v17 `UserType`/`GroupType`, `IAMAuthn` client, `CheckIfGlobalChangeAllowed` gate, v4 create/update insertion points, `GetIdentityValues` list semantics). Implemented `validateGlobalACPExternalIdentities` + tests, wired into v4 create + update. Bootstrap audit: no change needed. All build/test gates green. Not committed / not pushed.
