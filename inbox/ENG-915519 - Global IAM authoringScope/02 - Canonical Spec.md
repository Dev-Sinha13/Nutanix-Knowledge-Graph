---
tags: [eng-915519, spec, design]
status: locked-2026-06-01
---

# Canonical Spec — the field contract

The single source of truth for `authoringScope` semantics. All three repos MUST match this.

> **Design flip 2026-06-01:** field is now a scalar enum, not an array. The earlier "array for forward-extensibility" framing was dropped after reviewer pushback (Aditya, Praveen) since an entity is authored on exactly one product. The conversion landed across all three PRs in one coordinated push; details below.

## Field shape

| Property | Value | Notes |
|---|---|---|
| Type | Scalar enum string | `text` in Postgres, `string` in Go, `$ref` to the `AuthoringScope` enum model in OpenAPI |
| Enum values | `NC`, `PC` | Defined in `iamDefsDescriptions.yaml` via `DocRef`; also exposed as Go constants (`RoleRequestAuthoringScopeNC`, etc.) on the iam-utils generated models |
| Read-only | true | Clients cannot write; server stamps it. Lifted to the **model definition** in `authoring_scope.yaml` so the metadata propagates through bare `$ref` usages on the field site |
| Required | no | Omitted (`omitempty`) when empty string. Wire payload simply lacks the key |
| Immutable | yes | UPDATE statements deliberately skip this column |
| Filterable | yes (provisionally) | Equality filter on a scalar column — simpler than array containment. See [[05 - Cross-Repo Flow]] |

## Server-stamping rules

For a NEW entity being created via the API:

| Condition | Stamped value |
|---|---|
| `isGlobal=true` AND cluster product is PC | `"PC"` |
| `isGlobal=true` AND cluster product is NC | `"NC"` |
| `isGlobal=true` AND cluster product is something else (Xi, etc.) | `""` (omitted on the wire) |
| `isGlobal=false` | `""` (omitted on the wire) |

Implemented by `storage.ComputeAuthoringScope(isGlobal bool) string` — see [[07 - Handler Logic]].

## How the field looks across the layers

| Layer | Shape | Concrete |
|---|---|---|
| Public OpenAPI (ntnx-api-iam) | `$ref: AuthoringScope` on the field; `readOnly: true` on the enum model | `authoring_scope.yaml` defines the enum; `access_policy.yaml` and `roles.yaml` reference it |
| Internal API (iam-utils, themis.yaml) | `type: string` + inline `enum: ["NC", "PC"]` + `readOnly: true` + `x-omitempty: true` | Matches the `OperationSchemaChangeImpact` / `AccessPolicyType` precedent already in the file |
| Generated Go models (iam-utils) | `AuthoringScope string` with `json:",omitempty"` and a `validateAuthoringScopeEnum` validator | Hand-edited to mirror regen output for ONLY the AuthoringScope-specific portions — full regen pending Manish on go-swagger version pin |
| Storage struct (iam-themis) | `AuthoringScope string` with `json:",omitempty"` | Same wire-contract semantics as the public model |
| Postgres column | `authoring_scope text DEFAULT ''` | b-tree index for equality lookups |

## Lattice replication semantics

For global entities replicated across clusters via the lattice, `omitempty` on the Go struct's JSON tag means older peers that don't recognize the field simply don't see it (graceful forward-compat). New peers see and preserve the field. The conversion from array to scalar does not change this property — the empty-string zero value still gets omitted, same as nil-slice did.

## Why scalar, not array

Reviewers (Aditya on PR #910, Praveen on PR #1601) consistently pushed against the array shape with two arguments that proved decisive:

1. **Conceptual fit** — an entity is authored on exactly one product. Multi-authorship is not in the Kronos design and would require deeper architectural changes if it ever shows up.
2. **Operational simplicity** — equality filters (`authoring_scope = 'NC'`) are simpler to reason about than array containment (`authoring_scope @> ARRAY['NC']`), the b-tree index is cheaper than GIN, and the JSON wire shape is one character per value instead of three.

If a multi-origin requirement ever lands, it should be a separate field (e.g., `replicatedFrom: ["NC", "PC"]`) rather than overloading `authoringScope`. See [[10 - Decisions Log]] for the trail.

## See also

- [[01 - Overview]] for context
- [[06 - Storage & Migration]] for how this contract translates to Postgres
- [[07 - Handler Logic]] for how the value gets stamped
- [[10 - Decisions Log]] for the array-vs-scalar decision history
- Up: [[00 - Index]]
