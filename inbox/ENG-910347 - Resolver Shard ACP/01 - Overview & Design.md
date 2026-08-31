# Overview & Design

## The resolver model
Two gRPC services (proto/client in `iam-utils/identityresolver/`, servers in `iam-user-authn/server/identityresolver/`):
- **IdentityEnricher** (runs on NC): `Enrich(user/group UUIDs) -> IdentityEntry{connectorType, connectorIdentifier, userIdentifier, entityType}`. Turns local UUIDs into globally-identifiable properties.
- **IdentityResolver** (all replicas): `Resolve(IdentityEntry[]) -> {enrichmentKey -> local UUID}`. Maps properties back to a local UUID; generates a **deterministic v5 fallback UUID** when the entity is absent locally so references stay stable.

Canonical key: `connectorType|connectorIdentifier|userIdentifier|entityType` (`identityresolver/enrichment_key.go`). Per-RPC batch window `[1,50]`; iam-themis batches at configured `identityBatchSize`.

## The brownfield problem
A copied AP carries `UserList` / `UserGroupList` / `Identities` (UUIDs). Those are source-cluster-local; on the target the same humans/groups may have different UUIDs. greenfield->greenfield is a no-op (UUIDs already match); real work happens only for brownfield. Hence: **enrich on the producing (leader/NC) side, resolve on the consuming (follower) side.**

## Where this change sits (the gap)
#1535 wired enrich/resolve into **create/update ACP** (`v4_access_policy.go`) and **ApplyChange** (`apply_change.go`) — but NOT the shard-copy bootstrap. `git grep -i 'enrich|resolve' origin/master -- shard_data.go` -> 0 matches. A brand-new PC that bootstraps via `FetchShardData`/`WriteShardData` would persist unresolved cross-cluster UUIDs until a later ApplyChange happened to correct them. Closing that is this ticket.

## Approach (faithful extension, no forked logic)
1. Carry enriched identities in the shard batch payload (transit-only).
2. Enrich on the **leader** during `fetchAccessPoliciesBatch`.
3. Resolve on the **follower** during `WriteShardData`, before tenant adaptation + persistence.
4. Reuse the exact #1535 helpers (`enrichAPIdentities`, `resolveEnrichedIdentities`, `replaceACPIdentityUUIDs`).

The design doc requires enrichment metadata to be transit-only / stripped before persistence — satisfied: enriched entries live in a separate map (never on the AP row) and the map is nil'd after resolving.
