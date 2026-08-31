# ENG-910347 — Resolver RPCs for Shard read/write (ACP case)

> **Ticket: ENG-910347** — sibling work under the **ENG-910344** "RPC resolvers" effort.
> This vault folder is the distilled/curated record; the chronological blow-by-blow
> lives in-repo at `iam-themis/.notes/RESOLVER_SHARD_ACP_CONTEXT.md`.

## Status (2026-06-11, agent session 2)
- **Implemented + unit-tested**, NOT committed / NOT pushed.
- Lives in a **git worktree**: `/Users/dev.sinha/nutanix-core/iam-themis-shard-acp`, branch **`ENG-910347-shard-acp-resolver`** (tracks `origin/master` @ `f1be9cfa6`).
- All gates green on `master`: gofmt / build / vet / `go test` (6 new + full `lattice` pkg + `apihandler`).

## One-liner
Extend the enrich-on-source / resolve-on-target ACP identity handling — already shipped for the incremental **ApplyChange** path in **PR #1535** — to the **bulk shard-copy bootstrap** path (`FetchShardData` / `WriteShardData`), so a freshly-joined PC rewrites cross-cluster user/group UUIDs to local UUIDs instead of copying them verbatim.

## Key links
- Ticket: https://jira.nutanix.com/browse/ENG-910347
- Parent PR (MERGED 2026-06-04, `de3b0638`): https://github.com/nutanix-core/iam-themis/pull/1535
- Collision PR (OPEN, edits same `WriteShardData` block): https://github.com/nutanix-core/iam-themis/pull/1607 (ENG-811163 selective ACP persistence)
- Design doc shipped in #1535: `iam-themis/docs/global-acp-authn-uuid-resolution.md`

## Notes in this folder
- [[01 - Overview & Design]]
- [[02 - Implementation & Testing]]
- [[03 - Decisions Log]]
- [[04 - Open Questions & Status]]

## Repo scope
**`iam-themis` only.** No `ntnx-api-iam` schema change (internal lattice RPC, not a v4 wire field) and no `iam-utils` change (the `identityresolver` proto/client is already vendored). #1535 landed every cross-repo dependency.
