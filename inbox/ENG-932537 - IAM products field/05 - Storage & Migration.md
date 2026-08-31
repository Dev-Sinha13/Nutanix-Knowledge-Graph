---
tags: [eng-932537, storage, postgres, migration]
---

# Storage & Migration

## Postgres column

One column per entity table, identically shaped on `object` and `role` (NOT `access_policy` — out of scope per D24):

```sql
ALTER TABLE object  ADD COLUMN IF NOT EXISTS product_list text[] DEFAULT '{}';
ALTER TABLE role    ADD COLUMN IF NOT EXISTS product_list text[] DEFAULT '{}';
```

**Properties:**

- `text[]` (not jsonb) — native Postgres array type; OData `any(...)` predicates compile to `&&` overlap operator efficiently
- `DEFAULT '{}'` — existing rows get empty array on migration, not NULL. Treat empty as "not declared"; do not substitute a fallback at read time.
- `IF NOT EXISTS` — additive idempotent; safe to run on already-migrated clusters
- All migrations live in a single ENG-932537 block in `services/server/storage/sql/migrate.go` — they run as a single transactional unit at startup
- The `access_policy` table is **NOT** touched. The Q-L3 line `ALTER TABLE access_policy ADD COLUMN IF NOT EXISTS product_list text[] DEFAULT '{}';` was removed when the AP commit was dropped via rebase (D24).

## SQLite test schema mirror

SQLite has no native array type. The test schema in `services/server/storage/sql/client.go::beforeTestQueries` declares the columns as `text NULL`:

```sql
CREATE TABLE object (
  ...
  domain_uuids text [] DEFAULT '{}',
  product_list text NULL,
  ...
);
```

**The historical missing comma trap:** the user-introduced edit dropped the comma between `domain_uuids text [] DEFAULT '{}'` and `product_list text NULL`, which is a SQL syntax error in SQLite (`text [] DEFAULT '{}' product_list text NULL` parses as a column with two type modifiers). Caught + fixed in §15 Q-L3.

**Scanner handles both:** the `scanX` functions in `client.go` use `pq.Array(&x.ProductList)` which decodes Postgres `text[]` natively. When the column is omitted from the SELECT list, the scanner uses default `nil`. SQLite-backed tests treat the column as a text blob and rely on `omitempty` JSON marshalling to verify the field behaves correctly when absent.

## Why no GIN index (deferred)

The codebase has 10 GIN indexes on array columns in `role`, `access_policy`, `operation`, `authz_filter_search` — ALL of which are on authz hot paths (operation lookup, role-access enforcement). The `object` table has zero indexes of any kind.

Deferred per the user's "track all perf-related deferrals" directive. Lives as P1-P9 in the in-repo notes (`iam-themis/.notes/PRODUCTS_FIELD_CONTEXT.md` §12). Re-evaluate when:

- OData filter latency on `?$filter=products/any(...)` for entities exceeds 200ms p99 in production, OR
- A consumer (NC UI) reports slow page loads attributable to entity-list filtering

Add as `CREATE INDEX CONCURRENTLY ... USING GIN (product_list)` to avoid locking. See P5 for the CONCURRENTLY mandate rationale.

## Column-list discipline

The storage layer maintains multiple column-list constants for different SELECT shapes. ALL must include `product_list` for the field to be scannable. Object + Role only (AP omitted per D24):

| Constant | File | Purpose |
|---|---|---|
| `ObjectAllColumnsWithCount` | `storage/util.go` | v1 LIST entities (has `count(*) OVER()`) |
| `ObjectAllColumnsWithoutCount` | `storage/util.go` | v1 single GET entity |
| `V4ObjectAllColumnsWithCount` | `storage/util.go` | v4 LIST entities |
| `RoleAllColumnsForNonProxy` | `storage/util.go` | v1 / v4 role queries |
| `RoleAllColumnsForProxy` | `storage/util.go` | /proxy role queries |
| `RoleAllColumnsWithCount` | `storage/util.go` | LIST roles |

Field mapping constants for OData `$filter` and `$select` (translate API-facing `"products"` to storage-facing `"product_list"`):

| Constant | File | Direction |
|---|---|---|
| `ObjectFieldsToStorageFields` | `storage/util.go` | API → storage (used by `?$select=products`) |
| `ObjectStorageFieldsToObjectFields` | `storage/util.go` | storage → API (used by response builders + EDM) |
| `v4RoleFieldsToStorageFields` | `storage/util.go` | v4 API → storage for role |
| `v4RoleStorageFieldsToModelFields` | `storage/util.go` | v4 storage → API for role |

OData filter allow-lists (gate which fields the `$filter` parser will accept):

| Constant | File |
|---|---|
| `ObjectSupportedFilterFields` (`"products"`) | `storage/util.go` |
| `RoleSupportedFilterFields` (`"products"`) | `storage/util.go` |

The corresponding AccessPolicy constants (`AccessPolicyAllColumns*`, `v4APModelFieldsToStorageFields`, `APStorageFieldsToModelFields`, `V4AccessPolicySupportedFilterFields`, `APV4SupportedFilterFields`) are **untouched** — no `product_list` / `products` added; D24.

## SQL writes

INSERT and UPDATE statements use `pq.Array(...)` to bind Go `[]string` to Postgres `text[]`:

```go
"product_list": pq.Array(x.ProductList),
```

Notable callsites (added in this initiative — object + role only; D24 removed the AP callsite):

- `storage/sql/object.go` — `SeedObjects` (insert + update via map merge)
- `storage/sql/role.go` — `SeedRoles`, `UpdateRoleByName`

`pq.Array(nil)` correctly writes Postgres `'{}'` (empty array), not NULL — verified during the rollback gate run. So an entity loaded with `products: []` round-trips as `[]`, and one with `products` absent persists as `'{}'` and reads back as `nil`.

## See also

- [[02 - Canonical Spec]] for the field-shape contract this SQL implements
- [[06 - Handler Logic]] for who calls the writes (and when)
- [[09 - Open Questions & Followups]] for the GIN-index deferral details
- Up: [[00 - Index]]
