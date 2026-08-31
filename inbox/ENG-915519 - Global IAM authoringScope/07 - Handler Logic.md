---
tags: [eng-915519, handler, logic]
---

# Handler Logic — stamping, immutability, omit-on-empty

## The single source of truth: ComputeAuthoringScope

Lives in `iam-themis/services/server/storage/util.go`:

```go
// ComputeAuthoringScope returns the server-stamped authoring scope for a
// global entity per ENG-915519. It is populated only when isGlobal is true and
// the host runs as a known product (NC or PC); otherwise it returns nil so the
// field is omitted from serialization.
func ComputeAuthoringScope(isGlobal bool) []string {
    if !isGlobal {
        return nil
    }
    switch productutil.GetProductType() {
    case productutil.NcProduct:
        return []string{productutil.NcProduct}
    case productutil.PcProduct:
        return []string{productutil.PcProduct}
    default:
        return nil
    }
}
```

This function is the **only** place that computes the value. Both write paths call it:

1. The API → storage converter (when a client POST creates an entity)
2. The internal API → storage converter (for service-to-service traffic)

So we cannot accidentally end up with an inconsistent value across the two paths — both go through the same single function. This is the "defensive re-stamp" pattern.

## Immutability

UPDATE statements in `services/server/storage/sql/role.go` and `access_policy.go` deliberately OMIT `authoring_scope` from their `attrsMap`. Implications:

- Even if a downstream code path attempts to mutate it, the column never changes after INSERT
- No need for application-level "ignore client-supplied value" logic on the UPDATE path — the SQL just doesn't touch it
- The field is set ONCE at create time and is permanent for the lifetime of the row

A reviewer comment (T4) suggested adding a code comment to make this design intent explicit. Currently held out as a judgement call. See [[09 - Open Questions & Blockers]].

## Omit-on-empty

Storage struct definition:

```go
type Role struct {
    // ...
    AuthoringScope []string `json:"authoringScope,omitempty"`
}
```

Same for `AccessPolicy`. Same for the iam-utils generated request models.

Consequence:

- When nil or empty, the field is NOT included in JSON serialization
- Lattice peers running older code that don't recognize the field will simply not see it (graceful forward-compat)
- For non-global entities the value is always nil/empty, so they never show the field — by design

## v4 converter status — pending revendor

`services/server/apihandler/v4_access_policy.go` and `v4_roles.go` currently have comments like:

```go
// AuthoringScope intentionally not copied until upstream model revendor lands
```

The actual API request struct (vendored from `ntnx-api-iam`) doesn't have an `AuthoringScope` field yet because PR #910 hasn't merged. Once it merges and iam-themis revendors:

1. Remove those comments
2. Wire up `storageRole.AuthoringScope = storage.ComputeAuthoringScope(isGlobal)` — defensive re-stamp pattern. Even though the request struct will then have the field, we ignore the client-supplied value and always recompute server-side.

This is a follow-up commit on PR #1601 (or a successor PR) after the upstream lands.

## See also

- [[02 - Canonical Spec]] for the field contract
- [[05 - Cross-Repo Flow]] for the full write/read path
- [[06 - Storage & Migration]] for the SQL side
- Up: [[00 - Index]]
