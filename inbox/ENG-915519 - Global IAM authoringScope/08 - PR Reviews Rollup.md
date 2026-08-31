---
tags: [eng-915519, pr-reviews, rollup]
updated: 2026-05-28
---

# PR Reviews Rollup — all 22 review comments

Cross-PR inventory. Refresh with:

```bash
gh api repos/<owner>/<repo>/pulls/<num>/comments --paginate  # inline
gh pr view <num> --json comments                              # issue-level
```

Check both endpoints — they're distinct.

## Legend

- ✅ Fixed in code
- 🟢 Actionable now (small mechanical change)
- 🟡 Decided at design level, needs only a GitHub-side reply
- 🔵 Intentionally held out per "absolutely certain only" rule
- ⏳ Blocked on external dependency (Manish, SonarQube)
- ❌ Noise / no action

## PR #910 ntnx-api-iam (8 comments)

| # | Reviewer | File:Line | Comment | Status |
|---|---|---|---|---|
| 1 | aditya | `iamDefsDescriptions.yaml:443` | Don't put in single quotes | ✅ already fixed pre-session |
| 2 | aditya | `access_policy.yaml:60` | "Why is this an array" | 🟡 resolved by array decision; reply on GitHub |
| 3 | aditya | `access_policy.yaml:310` | Move enum to common file | ✅ done — `4cffeeb8` extract |
| 4 | aditya | `roles.yaml:31` | Doc has this as string for role | 🟡 array decision; verify Kronos doc, then reply |
| 5 | praveen | reply to #2 | "+1 should be a string" | 🟡 same as #2 |
| 6 | praveen | `access_policy.yaml:98` | Add to `x-filterable-properties` | 🟢 actionable now (array decided) |
| 7 | praveen | `roles.yaml:49` | Same as #6 for Role | 🟢 actionable now |
| 8 | dev-sinha | reply to #3 | Self-commitment to fix | informational |

## PR #357 iam-utils (5 comments)

| # | Reviewer | File:Line | Comment | Status |
|---|---|---|---|---|
| 9 | bot AI | `role_request.go:31` | Suggest `omitempty` | ⏳ blocked on go-swagger version pin |
| 10 | bot AI | `themis.yaml` | `minItems: 1, maxItems: 1` | 🟢 actionable now (decided: 0, 5); apply with regen |
| 11 | aditya | `access_policy_request.go:5` | Missing generated header | ⏳ blocked on go-swagger version pin |
| 12 | aditya | `role_request.go:5` | Same as #11 | ⏳ same |
| 13 | aditya | reply to #11 | "Let me know if these were not present upstream" | 🟢 just needs a reply |

## PR #1601 iam-themis (10 inline + 3 issue-level)

| # | Reviewer | File:Line | Comment | Status |
|---|---|---|---|---|
| T1 | bot AI | `apihandler/authoring_scope_test.go:3` | Test nil IsGlobal on PC | 🔵 held out (low value) |
| T2 | bot AI | `storage/authoring_scope_test.go:56` | Invalid product type test | 🔵 held out (user flagged optional) |
| T3 | bot AI | `storage/sql/authoring_scope_sql_test.go:469` | "No meaningful improvements" | ❌ junk; resolve as noted |
| T4 | bot AI | `storage/sql/role.go` | Immutability doc comment | 🔵 held out (judgement call) |
| T5 | praveen | `storage/sql/client.go:349` | "Should not be an array" | 🟡 array decision; reply |
| T6 | praveen | `storage/sql/client.go:382` | Same as T5 | 🟡 same |
| T7 | praveen | `storage/sql/migrate.go:1117` | Confusing migration comment | ✅ fixed — `e7ad49866` |
| T8 | praveen | `storage/sql/migrate.go:1124` | Discuss array + GIN with Manish | 🟡 array + GIN both kept |
| T9 | praveen | `storage/util.go:2532` | Remove Xi reference | ✅ fixed — `e7ad49866` |
| T10 | praveen | `storage/util.go:2536` | "Confirm and remove array" | 🟡 array kept |
| T11 | IamInfra | `role.go:73` (issue) | gofmt CI failure | ✅ fixed (real offender was `authoring_scope_sql_test.go`, not `role.go:73`) |
| T12 | IamInfra | (issue) | SonarQube Quality Gate FAILED, metrics empty | ⏳ needs investigation |
| T13 | bot | (issue) | "Generating code suggestions…" | ❌ noise |

## Disposition summary

- **5 fixed in code:** PR #910 #1 / #3; PR #1601 T7 / T9 / T11
- **9 unblocked by array decision** but need GitHub-side action: PR #910 #2 / #4 / #5 / #6 / #7; PR #1601 T5 / T6 / T8 / T10
- **3 hard-blocked on go-swagger version pin:** PR #357 #9 / #11 / #12
- **1 hard-blocked on SonarQube investigation:** PR #1601 T12
- **4 held out** per "absolutely certain only" rule: PR #1601 T1 / T2 / T4 + the holdout of #10 until regen
- **3 just need GitHub-side replies:** PR #357 #10 / #13; PR #910 #1 + others

## See also

- [[09 - Open Questions & Blockers]] for the unblocked / blocked categorization
- [[10 - Decisions Log]] for what decisions resolved which comments
- Up: [[00 - Index]]
