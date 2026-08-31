# Implementation & Testing

## Files changed (all in `iam-themis`, +91 lines source + 1 test)
| File | Change | Why |
|---|---|---|
| `services/server/apihandler/identity_enrichment.go` | Exported `EnrichAPIdentities` wrapper over unexported `enrichAPIdentities` | lattice can't call the unexported fn; lattice already imports apihandler (no cycle); avoids renaming ~26 call sites |
| `services/server/lattice/lattice_utils.go` | `APEnrichedIdentities map[string][]*pb.IdentityEntry` on `shardDataBatch` (`omitempty`) | Carries enriched identities leader->follower; **per-AP map** because a batch holds many APs (unlike single-AP ApplyChange); omitempty for fwd/bwd compat |
| `services/server/lattice/shard_data.go` | `enrichShardAPIdentities` called in `fetchAccessPoliciesBatch` (leader); `resolveShardAPIdentities` called in `WriteShardData` (follower) before adapt/persist; 2 helper methods | Enrich where source UUIDs are meaningful (NC); resolve where local UUIDs known; resolve before persist so raw cross-cluster UUIDs are never stored |
| `services/server/lattice/shard_identity_test.go` | NEW — 6 unit tests | Recreated (session-1 file was untracked, lost in stash) |

## Degrade behavior
Enrich failure aborts the fetch; resolve failure returns `kGenericAppError`. Both are **hard-fail / retryable**, mirroring the #1535 producer/consumer. Chosen over a verbatim-copy fallback to avoid silently persisting stale UUIDs.

## Testing — what IS covered
- 6 unit tests (fake Enricher + Resolver gRPC servers): enrich populates per-AP map / skips identity-less APs / propagates errors; resolve rewrites UUIDs + clears map / no-ops / propagates errors.
- Full `lattice` package + `apihandler` package pass; gofmt/build/vet clean.
- Helpers confirmed wired into the real RPC funcs (not just defined).

## Testing — gaps (honest)
1. No end-to-end `FetchShardData`->`WriteShardData` round trip (marshal of new field, pagination) — integration verified only by "compiles + wired in".
2. **No brownfield (different-UUID) scenario** — the actual point of the change. #1535's e2e ran identical UUIDs so it never exercised enrich/resolve.
3. No tavern / live-cluster / multi-PC run.
4. D4 tenant semantics unvalidated against a real cross-PC tenant.
5. No perf test of per-AP enrich (up to ~100 Enrich RPCs/page).

Highest-value next test: assemble a `shardDataBatch`, run `WriteShardData` against a fake resolver with **divergent** UUIDs, assert the persisted AP carries local UUIDs.



---

## Tavern e2e (update 2026-06-11)

**Correction:** there was never a "missing" tavern test — a thorough shard-copy e2e already exists at `api_tests/zzz_lattice_shard_copy/test_lattice_shard_copy.tavern.yaml` (added with the #1535 scaffolding). It already drives this change's path:

- ACPs are created **before** the learner joins the CG, so they replicate via shard copy (FetchShardData → WriteShardData), then are verified present on the learner.
- **ACP 6** = second user only → v5 UUID fallback on the learner.
- **ACP 8** = 10 identities → batched enrich/resolve (`identityBatchSize=2`).
- Authorize calls on the learner against the shard-copied identities.

**Gap I filled:** every existing ACP used only `user/uuid`; none carried a `userGroupList`. Added **ACP 9** (user + user-GROUP identity) with create / get / learner-verify / cleanup stages, so the **group** branch of `replaceACPIdentityUUIDs` is exercised through `WriteShardData`. Group identity syntax was lifted from `zz_global_iam/test_global_role_acp_crud.tavern.yaml`. YAML re-validated (131 stages parse).

**Honest limitation:** the tavern rig runs leader + learner against the *same* authn / Postgres, so UUIDs are identical and shard copy replicates **verbatim**. The tavern therefore proves wiring + no-regression (including the group path round-tripping) but **cannot** prove the actual cross-cluster UUID *rewrite*. Only a brownfield (divergent-UUID) rig validates the transform; that real coverage lives in the unit tests.
