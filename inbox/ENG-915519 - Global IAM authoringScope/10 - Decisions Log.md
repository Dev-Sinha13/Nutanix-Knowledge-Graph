---
tags: [eng-915519, decisions, log]
---

# Decisions Log

Chronological record of design and process decisions, with their consequences.

## 2026-06-01 ~13:00 PT — authoringScope flips from ARRAY to SCALAR ENUM (final)

**Decision:** Convert `authoringScope` from `array of enum string` to a `scalar enum string` across all three repos. Reverses the 2026-05-28 provisional "stay as array" decision.

**User directive:** "Ok I want you to convert authoringscope from an array to an enum." Single-line, no follow-up — the user had been weighing reviewer feedback for a few sessions and pulled the trigger.

**Mechanical changes that landed in this session:**

| Layer | Before | After |
|---|---|---|
| Public OpenAPI | `type: array` + `items: { $ref: AuthoringScope }` + `minItems: 0` + `maxItems: 5` + `readOnly: true` + `description: DocRef(authoringScopeDesc)` at the field site | Bare `$ref: AuthoringScope`. `readOnly: true` + `description: DocRef(authoringScopeDesc)` lifted onto the enum MODEL in `authoring_scope.yaml` so the metadata propagates through bare-ref usages |
| iamDefsDescriptions text | `["NC"]` / `["PC"]` syntax in `authoringScopeDesc` | `"NC"` / `"PC"` syntax |
| iam-utils themis.yaml | `type: array` + `items: { type: string, enum: ["NC", "PC"] }` + `minItems/maxItems` | `type: string` + inline `enum: ["NC", "PC"]` |
| iam-utils generated models | `AuthoringScope []string`; items-enum validator with strconv loop; `MinItems`/`MaxItems` checks | `AuthoringScope string`; prop-enum validator (single `validate.EnumCase` call); public constants `RoleRequestAuthoringScopeNC` / `PC` exposed |
| iam-themis storage struct | `AuthoringScope []string` | `AuthoringScope string` |
| iam-themis ComputeAuthoringScope | returns `[]string` (`nil`, `[]string{"NC"}`, `[]string{"PC"}`) | returns `string` (`""`, `"NC"`, `"PC"`) |
| Postgres column | `authoring_scope text[] DEFAULT '{}'` | `authoring_scope text DEFAULT ''` |
| Postgres index | `USING GIN (authoring_scope)` | b-tree (`(authoring_scope)`) |
| INSERT value | `pq.Array(role.AuthoringScope)` | `role.AuthoringScope` |
| Scan target | `pq.Array(&a.AuthoringScope)` | `&a.AuthoringScope` |
| sqlmock test rows | `"{NC}"` / `"{}"` | `"NC"` / `""` |

**Pushed commits (clean appends, not force-pushes):**

- ntnx-api-iam `ad2b392a` "ENG-915519: Convert authoringScope from array to scalar enum"
- iam-themis `b17451c59` "ENG-915519: Convert authoringScope from array to scalar string"
- iam-utils `573ca20` "ENG-915519: Convert authoringScope from array to scalar string enum"

**Verifications:**

- ntnx-api-iam: `mvn install -pl iam-api-external/iam-api-definitions -am` → BUILD SUCCESS
- iam-themis: gofmt clean, `go build ./services/...` clean, `go vet` only has pre-existing `proxy_role_test.go` warning, full storage + sql + apihandler test suites pass (including AuthoringScope-specific tests)
- iam-utils: `go build ./...` clean, `themisutil` test suites pass; pre-existing `circuitbreaker` vet warning unchanged

**Hand-edit note for iam-utils:** the generated Go models (`role_request.go`, `access_policy_request.go`) were hand-edited for ONLY the AuthoringScope-specific portions — full go-swagger regen still pending Manish on the version pin. The shape matches what regen will produce (mirrors the existing `AccessPolicyType` / `OperationSchemaChangeImpact` pattern in the same files), so the eventual regen should land as a minimal-diff sweep rather than a re-litigation.

**Resolves comments:** PR #910 #2 / #4 / #5, PR #357 #10 (cardinality portion; omitempty + header portions still blocked on Manish), PR #1601 T5 / T6 / T8 / T10.

**Still open:** reviewer ack needed across all three PRs (the change should be re-pinged so Aditya / Praveen can confirm the shape they pushed for has landed). Existing-row backfill question (T11 / [[09 - Open Questions & Blockers]]) is unaffected by the conversion but the backfill SQL is now substantially simpler (`UPDATE ... SET authoring_scope = 'PC' WHERE ... AND authoring_scope = ''`).

**What this supersedes:** the 2026-05-28 "stays as ARRAY (provisional)" entry below — that decision was provisional precisely because Manish hadn't weighed in and Aditya / Praveen were pushing back. The user's direct conversion call settles it without needing Manish's input.

## 2026-05-28 ~13:07 PT — authoringScope stays as ARRAY (provisional, SUPERSEDED 2026-06-01)

**Decision:** Treat `authoringScope` as an array of enum string, `minItems: 0, maxItems: 5`. NOT a scalar string.

**Context:** Four reviewers across three PRs (Aditya x2, Praveen x2, plus implicit "+1" reactions) pushed back asking for the field to be a string. Arguments:

- **For string** (reviewer push): only 2 enum values exist (NC, PC); one cluster has exactly one identity at any given time; "scope" is enum-like and singular by definition.
- **For array** (the working decision): forward-extensibility for future product types; parity with the `products` field (ENG-932537); allows multi-scope authoring if a federation pattern ever requires it.

**Status:** Provisional. Manish has not publicly weighed in. May flip later.

**Cascading consequences if it flips to string:**

- Column type: `text[]` → `text`
- Postgres default: `'{}'` → `''` or `NULL`
- Index: GIN → b-tree (T8 in [[08 - PR Reviews Rollup]])
- Go type on storage struct and request models: `[]string` → `string`
- IDL: `type: array, items: { enum: ... }` → `type: string, enum: ...`
- `ComputeAuthoringScope` return type changes
- JSON serialization shape changes (breaking for consumers)

**Resolves comments:** PR #910 #2 / #4 / #5, PR #357 #10 (partially), PR #1601 T5 / T6 / T8 / T10.

## 2026-05-28 ~13:13 PT — minItems: 0, maxItems: 5

**Decision:** Schema constraint values for the array.

- `minItems: 0` — empty array is valid (means "not declared")
- `maxItems: 5` — future headroom beyond the current 2 values

**Rationale:** matches the latest pushed state of PR #910's schema. Keeps all three repos aligned.

**Resolves comments:** PR #357 #10 (when regen lands).

## 2026-05-28 ~10:30 PT — Extract AuthoringScope enum to its own file

**Decision:** Move the `AuthoringScope` enum model out of `access_policy.yaml` into a new `authoring_scope.yaml` per Aditya's review #3. Both ACP and Role reference it via absolute `ModelRef({/namespaces/iam/versioned/v1/modules/authz/beta/models/AuthoringScope})`.

**Pushed as:** `4cffeeb8` on `ntnx-api-iam` `add-authoring-scope` branch.

**Resolves comments:** PR #910 #3, PR #910 #8.

## 2026-05-28 ~10:50 PT — iam-themis T7 / T9 / gofmt fixes

**Decision:** Apply only the "absolutely certain" fixes per user's rule:

- **T7** — drop the "Pattern mirrors the existing domain_uuids migration:" phrase in `migrate.go` (confusing reference); replace with self-contained description
- **T9** — delete the Xi paragraph in `util.go ComputeAuthoringScope` doc comment (Xi is scrapped)
- **gofmt** — fix the real offender (`authoring_scope_sql_test.go`, bullet conversion `*` → `-`), not `role.go:73` as IamInfra's bot reported (that complaint was stale; the bot fired on an earlier commit)

**Pushed as:** `e7ad49866` on `iam-themis` `ENG-915519-authoring-scope-handler` branch (after rebase onto `9afc64579` master).

**Resolves comments:** PR #1601 T7, T9, T11.

## 2026-05-28 — Hold-out list

**Decision:** Per user's "absolutely certain only" rule, deliberately NOT addressed in this session:

- PR #1601 T4 — bot-AI immutability code comment (judgement call; bot wording may not match user voice)
- PR #1601 T2 — invalid product type test (user said optional)
- PR #1601 T1 — bot AI test suggestion (low value)
- PR #1601 T3, T13 — bot AI noise
- PR #1601 T12 — SonarQube investigation (separate work)

**Resolves comments:** none yet; tracked in [[09 - Open Questions & Blockers]].

## Earlier work outside this session

(Sourced from `ntnx-api-iam/.notes/AUTHORING_SCOPE_CONTEXT.md`.)

- **2026-05-26** — Aditya granted write access on iam-utils
- **2026-05-24** — ntnx-api-iam minItems/maxItems lint passed; 8 violation entries removed, zero `authoringScope` mentions in any FAILED rule
- **Earlier** — Storage struct, ComputeAuthoringScope helper, v4 converter docs, SQL persistence, migration block all authored as part of original PR #1601 commit (`577fb531d`, now rebased to `2cf65eb45`)

## See also

- [[09 - Open Questions & Blockers]] for what's still open
- [[08 - PR Reviews Rollup]] for per-comment status
- Up: [[00 - Index]]
