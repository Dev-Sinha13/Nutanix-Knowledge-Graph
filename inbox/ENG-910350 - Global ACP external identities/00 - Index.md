# ENG-910350 — Global ACPs only on externally-managed identities

> **Ticket: ENG-910350** — part of the **IAM Entity Synchronization (Lattice) / Global IAM** epic.
> Sibling work: [[../ENG-910347 - Resolver Shard ACP/00 - Index|ENG-910347 (resolver shard ACP)]], [[../ENG-915519 - Global IAM authoringScope/00 - Index|ENG-915519 (authoringScope)]], [[../ENG-932537 - IAM products field/00 - Index|ENG-932537 (products)]].

## One-liner
When an Access Control Policy is created or updated as **global** (`isGlobal=true`) via the v4 API, reject it if any concretely-referenced user or group is **locally managed** (LOCAL user or SERVICE_ACCOUNT). Only externally-managed identities (SAML / LDAP / external IDP, and NC-propagated groups) may appear on a global, lattice-replicated ACP.

## Why
Global ACPs are lattice-replicated across clusters. Local users / service accounts are cluster-scoped and have no stable cross-cluster identity (cf. the enrich→resolve model in ENG-910347), so replicating an ACP that grants to a local identity is meaningless / a data-integrity hazard. This is an authoring-side guardrail on the NC (leader) cluster.

## Status
- Branch: **`ENG-910350-global-acp-external-identity`** (worktree `/Users/dev.sinha/nutanix-core/.wt-themis-global-acp`, tracks `origin/master` @ `734c90222`).
- See [[04 - Open Questions & Status]] for live state.

## Notes in this folder
- [[01 - Overview & Requirements]]
- [[02 - Implementation & Verification]]
- [[03 - Decisions Log]]
- [[04 - Open Questions & Status]]

## Repo scope
**`iam-themis` only.** No `ntnx-api-iam` schema change (`isGlobal` already exists; no new wire field). No `iam-utils` change (reuses the vendored `IAMAuthn` SDK client already used by the role-membership validator).
