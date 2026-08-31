# Open Questions & Status

## Status
- [x] Validator helper implemented
- [x] Wired into v4 create + update
- [x] Unit tests
- [x] Tavern API test (create + update reject a global ACP referencing a local user)
- [x] Bootstrap audit (default global ACPs referencing local/SA) — no change needed
- [x] Build / vet / test gates green
- Not committed / not pushed.

## Open questions
1. **Allow `EXTERNAL` (gmail-type) on global ACPs?** Currently allowed (D1). If product wants only tenant-IDP identities, drop `USERTYPE_EXTERNAL` from the allow-set.
2. **Wildcard `user:*` on a global ACP** — currently permitted (D4). Confirm with product whether a global grant-to-all should be blocked.
3. **v1 / proxy path (potential hole, not just out-of-scope).** Verified: the v1 `serveAccessPolicyPOST` → `buildAndCreateAuthorizationPolicy` path calls **neither** `AllowOperation`, `CheckIfGlobalChangeAllowed`, **nor** the new validator, and does not propose to the lattice. If a global ACP can be authored via v1/proxy, the guardrail is bypassed. The ENG-932537 products work explicitly treated v1 + /proxy as a real write path, so this needs confirmation — if v1 global create is reachable, extend the guardrail (and probably the missing `AllowOperation`/global-sync gate) there too.
4. **Batching** — validation does one `GetUserById` per distinct identity. ACP identity lists are small in practice; batch/list lookup is a possible optimization if large lists appear.
5. **Resolver-unreachable posture** — non-404 SDK errors surface as `InternalError` (fail-closed), matching ENG-910347 D3. Confirm this is the desired behavior vs. allowing the create when authn is transiently down.

## Gaps found in post-hoc review vs sibling tickets (2026-07-09)
- [x] **In-repo `.notes/` context file** — created `iam-themis/.notes/GLOBAL_ACP_EXTERNAL_IDENTITY_CONTEXT.md` (sibling convention: vault is distilled, in-repo file is chronological log).
- [x] **Tavern API test** — added 4 stages to `api_tests/zz_global_iam/test_global_role_acp_crud.tavern.yaml`: create a LOCAL authn user, assert global-ACP **create** referencing it fails (400/IAM-20027), assert global-ACP **update** adding it beside the AD user fails (400/IAM-20027), delete the local user. LDAP-user allow path already exercised by the existing combo stages. ASCII-only verified; YAML parses (65 stages). Message asserted `!anystr` (IAM-20027 covers both "locally-managed" and "unresolvable"). Not yet run on a cluster.
- [ ] **`docs/` design doc** — global-IAM/lattice changes conventionally ship one (e.g. `global-acp-authn-uuid-resolution.md`). None written.
- Reused-not-reinvented: gate (`CheckIfGlobalChangeAllowed`), lookup pattern (`v4_role_membership_validator.go`), error mapper (`mapIAMAuthnSDKError`) — consistent with sibling reuse discipline.

## Reproduce
- Worktree: `/Users/dev.sinha/nutanix-core/.wt-themis-global-acp`, branch `ENG-910350-global-acp-external-identity` (base `origin/master` @ `734c90222`).
