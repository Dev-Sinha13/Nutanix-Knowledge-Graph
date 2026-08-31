---
tags: [eng-932537, iam, products, kronos, moc]
status: in-progress
created: 2026-05-29
updated: 2026-05-29
---

# ENG-932537 — IAM `products` Field

> A read-only multi-valued field on IAM Entities and Roles that records the originating product context (NC / PC / NCM). Surfaced on v4 read APIs, v1 entity read+write, and /proxy entity write. Enables Nutanix Central topology-aware filtering across federated clusters. **AccessPolicies are explicitly out of scope** (was wired in Q-L3, then reverted 2026-05-29 — see [[08 - Decisions Log]] D24).

## Current state at a glance

- **4 PRs open**: `ntnx-api-iam#914`, `iam-utils#358`, `iam-themis#1603`, `iam-bootstrap#768`
- **5th PR pending**: parallel iam-utils swagger PR for `ClientObject.products` (draft in [[12 - PR Description Drafts]])
- **Branch state** (iam-themis): `7aa4c7c77` on `ENG-932537-products-field`, force-pushed to origin. Three commits on `master`: `7895a1e8f` (entity + role v4) → `ec87ae6a4` (v1/proxy entity surfacing, post-AP-rollback rebase) → `7aa4c7c77` (ASCII regression scrub on role tavern, D25)
- **ntnx-api-iam**: `01fb9033` on `ENG-932537-products-field`, force-pushed. Schema covers Entity + Role only (AP yaml + description block reverted 2026-05-29)
- **Scope landed**: entities + roles on v4; entities on v1 + /proxy (read + write)
- **Scope deferred**: roles on v1 + /proxy; runtime `ProductAllowedTypesMap` validation in iam-themis
- **Scope removed (D24)**: AccessPolicies on all surfaces. Was wired (commit `a411a8943` in iam-themis, AP yaml in ntnx-api-iam `658bbe23`); rolled back per user direction. Reflog-reachable for forensics if ever reopened.
- **Hard blocker**: CI "IAMv2 Automation Tests" red on #1603 and #768 — two competing hypotheses now: (A) cross-repo coordination, (B) the ASCII regression that was on origin since session 2 (fixed by `7aa4c7c77` D25; if IAMv2 turns green on the new tip, B was the culprit)
- **Soft open**: all 4 PR bodies still GitHub template placeholders; drafts in [[12 - PR Description Drafts]] need copy-paste

## Read in this order

1. [[01 - Overview]] — what `products` is, why it matters, how it differs from `authoringScope`
2. [[02 - Canonical Spec]] — the field contract (shape, allowed values, semantics per entity type)
3. [[03 - Repository Map & Flow]] — 4 repos, who owns what, data flow from `objects.json` to API response
4. [[04 - PR Status]] — live state of all 4 PRs, CI, merge ordering
5. [[05 - Storage & Migration]] — Postgres `text[]`, SQLite test-mirror, additive migrations
6. [[06 - Handler Logic]] — derivation chokepoints (entity seeding, role union)
7. [[07 - API Surface Matrix]] — entity/role × v1/v4/proxy read+write status (the at-a-glance scope view)
8. [[08 - Decisions Log]] — D1–D24 grouped by epoch (bootstrap, AP wiring, v1/proxy widening, AP rollback)
9. [[09 - Open Questions & Followups]] — what's left to do and what was deliberately deferred
10. [[10 - Process Lessons]] — reusable patterns (TEMP HACK pairing, commit-tree surgery, ASCII discipline)
11. [[11 - Issues Encountered & Fixes]] — specific incidents from this work with symptom → cause → fix → lesson
12. [[12 - PR Description Drafts]] — parking lot for the 5 PR bodies (copy-paste ready)

## Where the code-side context lives

In-repo work log: `iam-themis/.notes/PRODUCTS_FIELD_CONTEXT.md` (17+ sections, ~1200 lines, blow-by-blow execution detail per session). That file is the chronological / append-only record; this vault is the distilled / curated re-organization.

Canonical cursor rule: `.cursor/rules/iam-products-field.mdc` (always-applied; documents the current field contract — AP out-of-scope after D24 — plus logging discipline).

## Quick links to the PRs

- ntnx-api-iam #914 (v4 schema, Entity + Role only): https://github.com/nutanix-core/ntnx-api-iam/pull/914
- iam-utils #358 (configload + maps): https://github.com/nutanix-core/iam-utils/pull/358
- iam-themis #1603 (handlers + storage): https://github.com/nutanix-core/iam-themis/pull/1603
- iam-bootstrap #768 (seeding): https://github.com/nutanix-core/iam-bootstrap/pull/768

## Related work

- Sibling Kronos initiative: [[../ENG-915519 - Global IAM authoringScope/00 - Index|ENG-915519 — authoringScope]]. Independent field, same federation rationale, much overlap in repo layout and review process patterns.
