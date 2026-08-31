# Overview & Requirements

## Requirement (from ticket + user guidance 2026-07-08/09)
- "Global ACPs only allowed on externally managed users." Enforced **from the API, at ACP create** (and update).
- **Externally managed (ALLOWED):** IDP (SAML/OIDC) users, AD/LDAP users, and user groups synced from those external sources.
- **Locally managed (REJECTED):** standard local users and service accounts (SAs). Strictly-local user groups are likewise rejected.
- Validate **both** explicit identity lists (the concrete UUIDs in the identities filter) and reject filters that pin to a concrete local identity. Purely dynamic filters (e.g. wildcard `*`) are left to per-cluster runtime evaluation.
- `isGlobal` is an existing property → **no API schema change**.

## User-type source of truth
The `IAMAuthn` SDK client (`ntnx-api-golang-sdk-internal/iam-go-client/v17`) exposes the type on the entity models returned by `GetUserById` / `GetUserGroupById`:

- `authn.User.UserType *authn.UserType`, enum: `USERTYPE_{UNKNOWN,REDACTED,LOCAL,SAML,LDAP,EXTERNAL,SERVICE_ACCOUNT,PROPAGATED}`.
- `authn.UserGroup.GroupType *authn.GroupType`, enum: `GROUPTYPE_{UNKNOWN,REDACTED,SAML,LDAP,PROPAGATED}`.

Note: there is **no OIDC user type** in the v4 enum — OIDC is represented as SAML — so the deny/allow decision is expressed over the enum values that actually exist.

## Where global ACP authoring happens
The v4 handler `services/server/apihandler/v4_access_policy.go` is the entry point. Both create and update branch on `isGlobal`, and the global branch already:
1. Builds / processes the storage object (populating `UserList` / `UserGroupList` from the identities filter).
2. Gates on `utils.CheckIfGlobalChangeAllowed(apiCtx)` (NC-product + `GlobalSyncEnabled` config / `IsGlobalSyncEnabled()` feature flag / ZK node).
3. Enriches identities and proposes the change to the lattice.

The new validation slots in **between step 2 and step 3** — so it runs only for global ACPs when global sync is allowed, and before anything is proposed to the lattice.

## See also
- [[02 - Implementation & Verification]] — exact code, insertion points, gates.
- [[03 - Decisions Log]] — deny/allow-set, wildcard handling, v4-only scope.
