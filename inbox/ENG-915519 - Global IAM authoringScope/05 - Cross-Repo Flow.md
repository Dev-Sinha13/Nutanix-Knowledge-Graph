---
tags: [eng-915519, architecture, flow]
---

# Cross-Repo Flow — how authoringScope moves through the system

The field flows from public schema → internal API → handler → storage → response. Each repo owns one layer; the contract between layers is [[02 - Canonical Spec|the canonical spec]].

## Write path (entity creation)

```
Client (POST /api/iam/v4/.../roles)
      ↓ (request body — authoringScope IGNORED if present, it's read-only)
ntnx-api-iam schema: validates request matches IDL
      ↓ (deserialized into Go request struct from iam-server-codegen vendor)
iam-themis v4 handler (v4_roles.go / v4_access_policy.go)
      ↓ (currently does NOT copy authoringScope from request — pending
         upstream revendor; see [[04 - PR Status]] follow-up note)
storage.Role / storage.AccessPolicy
      ↓ (handler invokes ComputeAuthoringScope(isGlobal))
storage struct's AuthoringScope []string is set
      ↓ (handler routes to internal API for persistence)
iam-utils RoleRequest / AccessPolicyRequest (themisutil/generated/models)
      ↓ (HTTP call to internal service)
iam-themis storage layer FromRoleRequest / FromAPItoStorageAccessPolicy
      ↓ (re-stamps via ComputeAuthoringScope — defensive; single source of truth)
iam-themis SQL INSERT (role.go / access_policy.go)
      ↓ (attrsMap includes "authoring_scope": pq.Array(role.AuthoringScope))
Postgres role.authoring_scope text[] / access_policy.authoring_scope text[]
```

## Read path

```
Client (GET /api/iam/v4/.../roles/{id})
      ↓
iam-themis v4 handler
      ↓ (SELECT includes authoring_scope per RoleAllColumnsWith* constants)
iam-themis SQL scan (scanRole / scanAccessPolicy)
      ↓ (populates storage.Role.AuthoringScope)
storage → API response object
      ↓ (response serializer omits the field when nil/empty due to omitempty)
Client receives: { "authoringScope": ["PC"], ... } OR no field at all
```

## Filterability — the two-side wiring

To support `$filter=authoringScope eq 'PC'` end-to-end, **two edits** are required (mirrors the `products` pattern in ENG-932537):

1. **Schema side** — `x-filterable-properties` in `access_policy.yaml` / `roles.yaml` in `ntnx-api-iam`. Generates EDM binding metadata in the vendored Go models.
2. **Storage side** — append `"authoringScope"` to `V4AccessPolicySupportedFilterFields` and `V4RoleSupportedFilterFields` in `iam-themis/services/server/storage/util.go`.

Schema side is PR #910 #6 / #7 (pending small work). Storage side is NOT in PR #1601 yet — needs a follow-up.

## Immutability path

UPDATE statements in `role.go` and `access_policy.go` deliberately OMIT `authoring_scope` from their `attrsMap`. So even if downstream code attempts to mutate it, the column never changes after the initial INSERT.

See [[07 - Handler Logic]] for the rationale and how this enforces server-stamped semantics without application-level "ignore client value" logic on the UPDATE path.

## Lattice replication path

For global entities, the storage struct's `omitempty` JSON tag means:

- Older lattice peers that don't recognize the field will simply not see it (forward-compat)
- New peers see and preserve the field
- Non-global entities never have the field present in the wire format anyway

## See also

- [[03 - Repository Map]] for which repo owns which layer
- [[06 - Storage & Migration]] for the SQL details
- [[07 - Handler Logic]] for the ComputeAuthoringScope and immutability rules
- Up: [[00 - Index]]
