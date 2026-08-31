# Decisions Log

- **D0:** "Resolver RPCs" = `IdentityEnricher` / `IdentityResolver` (iam-utils/identityresolver), NOT the enforcement `AssociationResolver`. Evidence: `identity_resolver.proto`.
- **D1:** Enrich on the leader during Fetch (natural mirror of ApplyChange's enrich-at-originating-request; shard copy has no other producer hook).
- **D2:** Added exported `EnrichAPIdentities` wrapper rather than renaming `enrichAPIdentities` across ~26 call sites.
- **D3:** Hard-fail (retryable) on enrich/resolve outage; no verbatim-copy fallback (avoid persisting stale cross-cluster UUIDs).
- **D4 (OPEN — needs reviewer):** Resolve using the AP's received `TenantID` pre-adaptation, mirroring the ApplyChange consumer. Cross-PC tenant basis flagged for a reviewer who owns the federation-tenant model. v5 fallback keeps refs deterministic regardless.
- **D5 (DECIDED):** Keep per-AP enrich; defer cross-AP batching (collect/dedupe UUIDs per page -> fewer Enrich calls -> demux) until the perf testing praveenav-23 requested on #1535. Faithful to the reviewed per-AP shape, already tested.
- **D-branch / D6 (SUPERSEDED):** Originally based on `origin/ENG-910344` as a follow-up. #1535 merged -> base is now `master`.
- **D7 (OPEN — needs user):** Delivery + #1607 sequencing. (a) base on `master`, resolve the trivial `WriteShardData` conflict if #1607 merges first; or (b) stack on `ENG-811163-selective-acp-persistence`. Semantics compose (filter -> resolve) either way; only the text conflicts.
- **D8:** Operate in a dedicated **git worktree** off `master` rather than switching the shared checkout, because parallel agents keep flipping its branch (seen on ENG-915519 then ENG-932537).
