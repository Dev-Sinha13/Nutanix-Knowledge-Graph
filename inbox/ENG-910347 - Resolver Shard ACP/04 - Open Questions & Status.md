# Open Questions & Status

## Open (need input)
1. **D7 — delivery / #1607 sequencing.** PR #1607 (selective ACP persistence, OPEN) inserts `filterAccessPoliciesForSelf(...)` into the same `WriteShardData` AP block. Correct follower order is **filter (drop out-of-scope) -> resolve (survivors only) -> adapt -> persist**; my resolve already sits after that filter point, so semantics compose but text conflicts. Choose: base on `master` (rebase when #1607 lands) vs. stack on #1607's branch.
2. **Integration test for the brownfield case** — add it or not? It's the only meaningful end-to-end validation; unit tests are current coverage.
3. **D4 — tenant basis** — reviewer confirmation that resolving with the received `TenantID` is correct cross-PC.

## Status snapshot
- Code implemented + unit-tested in worktree `iam-themis-shard-acp` (branch `ENG-910344-shard-acp-resolver-v2`).
- Nothing committed or pushed (no greenlight yet).
- All local gates green on `master`.

## Contacts
- **Manish Lokur** (`manishlokur`) — author of #1535 + #1607; primary contact for sequencing.
- **praveenav-23** — #1535 reviewer; flagged batch-size perf testing (relates to D5).
- **aditya** — #1535 reviewer; "use AP not ACP" naming (already satisfied).

## Immediate next steps (when greenlit)
1. Decide D7.
2. (optional) Add the brownfield integration test.
3. Commit on the worktree branch with a WHY-focused message; open PR cross-referencing #1535 (parent) + #1607 (sequencing).
