---
tags: [eng-932537, overview]
status: stable
---

# Overview — what `products` is and why

## What

`products` is a **read-only multi-valued field** (Go `[]string`, Postgres `text[]`) on two IAM entity types:

- **Entities** (`storage.Object`) — the product context the entity *belongs to*, seeded per cluster from `objects.json`
- **Roles** (`storage.Role`) — server-computed union of `products` across every entity the role's operations act on

> **Out of scope:** AccessPolicies. Q-L3 wired AP support in iam-themis + ntnx-api-iam, then it was reverted 2026-05-29 per user direction (D24). The reflog still carries the prior implementation if NC ever needs AP-level products filtering.

Allowed values are gated by the cluster's running product type (see [[02 - Canonical Spec]] for the `ProductAllowedTypesMap`):

| Cluster product type | Allowed `products` values |
|---|---|
| PC clusters | `["PC"]` |
| NC clusters | `["NC", "NCM"]` |

Explicitly NOT allowed: `AHV`, `CALM`, `UNKNOWN`, `Xi`. `Xi` exists as a `productutil` constant but is not in the allow-map.

## Why

Part of the **Global IAM / Nutanix Central** initiative. Specifically:

- Lets Nutanix Central (NC) UI **filter IAM entities by originating product** when displaying federated views
- Provides a stable per-entity product tag that survives lattice replication across clusters (`omitempty` for forward-compat with older peers)
- Closes the gap that previously left NC unable to distinguish which underlying product an entity / role / AP originated from

Without this field, NC has no way to know which cluster product authored or applies to an IAM entity in a federated environment; it can only list everything globally.

## What it's NOT

- **NOT the same as `authoringScope`** — see the sibling [[../ENG-915519 - Global IAM authoringScope/00 - Index|ENG-915519]]. `authoringScope` records *who authored* an entity (a single cluster's identity at creation time); `products` records *which products the entity applies to* (an open-ended product-context set). Different fields, different semantics, different mutability rules.
- **NOT user-settable.** The API rejects (or ignores) any client-supplied `products` on requests; values are server-computed or seeded.
- **NOT keyed off `isGlobal`.** A non-global role can still have `products: ["NC", "PC"]` if its operations touch entities tagged that way. Authoring/edit-permission gating is governed by `isGlobal` + `authoringScope` (ENG-915519), not by `products`.
- **NOT validated at runtime** against `ProductAllowedTypesMap` in iam-themis. Validation lives only at bootstrap config-load time (in `iam-utils/configutil`). See D21 in [[08 - Decisions Log]].
- **NOT a property of AccessPolicies.** Was wired in Q-L3, then reverted (D24). If NC ever needs AP-level filtering, see the reflog SHAs in §17 of `iam-themis/.notes/PRODUCTS_FIELD_CONTEXT.md`.

## Who's involved

- **Manish Lokur** — Original author of the parallel `ENG-932537` (no-suffix) branches across all repos. His prior implementation was either superseded or branched-off; see D1 in [[08 - Decisions Log]] for the iam-bootstrap branch-reuse rejection rationale.
- **Aditya** — Primary reviewer on ntnx-api-iam and iam-utils (per ENG-915519 pattern; reviews not yet landed on #914 / #358)
- **Praveen** — Primary reviewer on iam-themis (per ENG-915519 pattern; reviews not yet landed on #1603)
- **Dev (you)** — Implementing across all 4 repos

## See also

- [[02 - Canonical Spec]] for the formal field contract
- [[06 - Handler Logic]] for how the server actually computes/stamps the value per entity type
- [[07 - API Surface Matrix]] for which APIs surface the field today
- Up: [[00 - Index]]
