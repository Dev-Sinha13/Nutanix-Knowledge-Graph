---
tags: [eng-915519, storage, migration, postgres]
status: scalar-as-of-2026-06-01
---

# Storage & Migration — DB column, b-tree index, backfill question

> **Updated 2026-06-01:** column is `text` (scalar), not `text[]` (array). Index is b-tree, not GIN. Default is `''`, not `'{}'`. The migration runs cleanly because the field hasn't been deployed to any real DB yet — the array version of the migration in earlier commits is replaced before the migration ever fires in production.

## The migration block

Lives in `iam-themis/services/server/storage/sql/migrate.go`:

```go
{
    // ENG-915519: authoringScope on global ACPs and Roles. Adds a text
    // column with empty-string default, plus a b-tree index for
    // equality lookups like `authoring_scope = 'NC'`.
    stmt: `
        ALTER TABLE role          ADD COLUMN IF NOT EXISTS authoring_scope text DEFAULT '';
        ALTER TABLE access_policy ADD COLUMN IF NOT EXISTS authoring_scope text DEFAULT '';
        CREATE INDEX IF NOT EXISTS idx_role_authoring_scope          ON role          (authoring_scope);
        CREATE INDEX IF NOT EXISTS idx_access_policy_authoring_scope ON access_policy (authoring_scope);
    `,
},
```

## Properties of the migration

- **Idempotent** — `IF NOT EXISTS` on both column add and index create. Safe to re-run.
- **Performance-safe** — `ADD COLUMN` with a constant default is metadata-only in Postgres 11+ (no table rewrite, no full-table ACCESS EXCLUSIVE lock). B-tree index creation on a brand-new empty column is instant.
- **Default `''`** — empty string, not NULL. Existing rows get this value on first migration run. `omitempty` on the Go struct ensures the JSON wire shape omits the field for these rows.
- **B-tree choice** — correct for equality lookups (`authoring_scope = 'NC'`). The earlier array version used GIN for `@>` containment; with the scalar flip, b-tree is the natural and cheaper choice. See [[02 - Canonical Spec]] for the full reasoning.

## When the migration actually runs

It runs **automatically** at iam-themis service startup, against whichever Postgres the service connects to. The migration runner walks the ordered `migrations []migration` slice in `migrate.go` and applies any new entries that haven't yet been recorded as applied. **No manual step required.**

## SQLite bootstrap (test path)

`iam-themis/services/server/storage/sql/client.go` also has the column definition in its bootstrap CREATE TABLE statements (for the in-process SQLite path used by integration tests). That was updated in lockstep with `migrate.go` to `text DEFAULT ''`.

## Open question — existing-row backfill

The migration as-is leaves **existing global** ACPs and Roles with `authoring_scope = ''`. Day 1 after deploy, `$filter=authoringScope eq 'PC'` returns zero matches for pre-migration data.

Two paths:

| Path | Behavior | Implementation cost |
|---|---|---|
| **No backfill (current PR)** | Old rows fill in gradually as they're touched via the API | Zero — already implemented |
| **Cluster-aware backfill** | Old global rows get the current cluster's scope immediately | Needs a Go-coded migration entry since the value depends on `productutil.GetProductType()` at runtime |

Backfill sketch (now much simpler with scalar):

```sql
-- after the ALTER TABLE statements, conditional on current cluster:
UPDATE role          SET authoring_scope = 'PC'  -- on PC cluster
   WHERE is_global = true AND authoring_scope = '';
UPDATE access_policy SET authoring_scope = 'PC'
   WHERE is_global = true AND authoring_scope = '';
```

The scalar shape makes this notably cleaner than the array version (no `cardinality(...)` or `ARRAY[...]` literal needed). This is a product decision, not a code blocker. Tracked in [[09 - Open Questions & Blockers]]. A good follow-up question to ask: how was this handled for `domain_uuids` when it was added?

## Non-global rows are explicitly not in scope

Non-global tenant entities correctly get `''` on the migration and stay that way forever — that's by design. Only globals need an authoring scope.

## What changed in the scalar conversion

The diff vs the array version (for the next agent reading this):

| Component | Was | Is |
|---|---|---|
| Migration column type | `text[] DEFAULT '{}'` | `text DEFAULT ''` |
| Migration index | `USING GIN (authoring_scope)` | `(authoring_scope)` (b-tree) |
| Bootstrap CREATE TABLE | `authoring_scope text [] DEFAULT '{}'` | `authoring_scope text DEFAULT ''` |
| INSERT SQL value | `pq.Array(role.AuthoringScope)` | `role.AuthoringScope` |
| Scan target | `pq.Array(&a.AuthoringScope)` | `&a.AuthoringScope` |
| sqlmock test row value | `"{NC}"` | `"NC"` |
| sqlmock empty value | `"{}"` | `""` |

The conversion is mechanical at this layer — the storage abstraction barely cared about the array-ness, it was mostly the `pq.Array(...)` wrapper that disappeared.

## See also

- [[02 - Canonical Spec]] for the field contract
- [[07 - Handler Logic]] for how new entities get their value stamped
- [[09 - Open Questions & Blockers]] for the backfill decision
- [[12 - Issues Encountered & Fixes]] for prior `pq.Array` / GIN troubleshooting (now obsolete)
- Up: [[00 - Index]]
