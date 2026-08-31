---
tags: [eng-915519, iam, kronos, moc]
status: in-progress
created: 2026-05-28
updated: 2026-06-01
---

# ENG-915519 — Global IAM authoringScope

> A read-only field on global ACPs and Roles that records the originating product (NC or PC). Part of the Kronos / Global IAM initiative.

## Current state at a glance

- **3 active PRs**, all OPEN: `ntnx-api-iam#910`, `iam-utils#357`, `iam-themis#1601`
- **Design locked (2026-06-01):** **scalar enum** `{NC, PC}`, server-stamped, immutable, omit-on-empty. Reverses the prior provisional "array" call. Conversion landed across all three PRs in one coordinated push — see [[10 - Decisions Log]] for the trail and [[02 - Canonical Spec]] for the new contract.
- **Hard blocker (unchanged):** `go-swagger` version pin (Manish) — blocks iam-utils regeneration; the scalar conversion is hand-edited around it for now.
- **Soft open:** existing-row backfill (`''` for now vs cluster-aware UPDATE). Simpler with scalar than with array — see [[06 - Storage & Migration]].
- **Action needed:** re-ping reviewers (Aditya on #910, Praveen on #1601) so the disagreement that drove the conversion can be officially resolved on GitHub.

## Read in this order

1. [[01 - Overview]] — what this is and why it matters
2. [[02 - Canonical Spec]] — the design contract
3. [[03 - Repository Map]] — which repo owns what
4. [[04 - PR Status]] — live state of the three PRs
5. [[05 - Cross-Repo Flow]] — how the field propagates
6. [[06 - Storage & Migration]] — DB column, GIN index, backfill question
7. [[07 - Handler Logic]] — ComputeAuthoringScope, immutability, omit-on-empty
8. [[08 - PR Reviews Rollup]] — all 22 review comments + disposition
9. [[09 - Open Questions & Blockers]] — what's pending decisions / external dependencies
10. [[10 - Decisions Log]] — chronological record of what's been settled
11. [[11 - Process Lessons]] — abstract meta-lessons (verification ladder, question-framing template, gotchas)
12. [[12 - Issues Encountered & Fixes]] — concrete postmortem: specific incidents → causes → fixes → lessons

## Where the code-side context lives

The companion progress doc in the repo: `ntnx-api-iam/.notes/AUTHORING_SCOPE_CONTEXT.md` (17 sections, finer granularity than these notes).

## Quick links to the PRs

- ntnx-api-iam: https://github.com/nutanix-core/ntnx-api-iam/pull/910
- iam-utils: https://github.com/nutanix-core/iam-utils/pull/357
- iam-themis: https://github.com/nutanix-core/iam-themis/pull/1601

## Related work

- Sibling Kronos initiative: [[../ENG-932537 - IAM products field/00 - Index|ENG-932537 — IAM `products` field]] (different field, different mutability — `authoringScope` is who-authored / immutable; `products` is what-it-applies-to / server-derived per-entity-type)
