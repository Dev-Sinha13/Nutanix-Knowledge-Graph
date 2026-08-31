---
tags: [eng-915519, lessons, process, meta]
---

# Process Lessons — meta-lessons from working through this

Not about authoringScope itself; about how to navigate a multi-repo, multi-PR feature like this effectively. These came out of the working sessions and are captured here so future tickets benefit.

> **Companion note:** [[12 - Issues Encountered & Fixes]] has the concrete incidents (symptoms, causes, fixes) that these abstract lessons were derived from. This note is the wisdom; that note is the receipts.

## Information organization strategies that worked

1. **One canonical progress doc** — `ntnx-api-iam/.notes/AUTHORING_SCOPE_CONTEXT.md` — maintained as a live shared-context file. Numbered sections; corrections explicit (the doc was wrong twice in a session and the corrections are preserved with `> Correction note:` markers so the next reader doesn't repeat the same wrong reads).
2. **A "what I'm asking about / not asking about" frame** for every external question — see the question-framing template below.
3. **Inverted-pyramid structure for snapshots** — headline state first, then context. Saves the next reader from wading through history they don't need.
4. **Decision logs separate from open questions** — collapses cognitive load. If it's decided, it's not open; if it's open, it's not yet decided.
5. **"Where I am in the conversation" section** updated at the end of every session — explicit cursor that survives session boundaries.

## Question-framing template (for Glean / colleagues / future agents)

When asking a "should I do X or Y" question, hit these in order:

1. **One-line headline with the canonical identifier** (ticket, RFC number, file path) — anchors retrieval
2. **One paragraph of background** — what the change is, who it's for, state of related work
3. **The specific decision point as a clean either/or** — Option A vs Option B, with observable consequences of each
4. **Targeted sub-questions** — the actual unknowns you want resolved
5. **Out-of-scope statement** — what's already decided, so the responder doesn't re-litigate
6. **Optional: prior-art pointers** — "did we settle this for X column / Y feature?"

**Biggest mistake to avoid:** dumping raw narrative context and trailing off with "thoughts?" — leaves the responder unsure which decision to weigh in on.

## Multi-repo gotchas learned

### Branch drift across shells

When a parallel agent is operating on the same repo, `git switch` may not persist as expected — the other agent may switch back behind your back. **Always run `git rev-parse --abbrev-ref HEAD`** at the start of any significant operation.

### Sticky `go.mod` local-replace blocks

When one branch has `replace ... => ../sibling-repo` directives in `go.mod` for local dev, running `go build` / `go test` will mutate `vendor/` to match the local siblings. Each subsequent build silently re-contaminates after every revert.

**The fix:** chain `git reset --hard HEAD` → action → verify in ONE shell call to minimize the contamination window. Or stop the parallel agent first.

### Force-push with `--force-with-lease`

After rebasing a feature branch, you need a force-push to update remote. **Always use `--force-with-lease`** — it aborts if anyone else pushed in the meantime. Plain `--force` is dangerous; `--force-with-lease` is the standard safe variant.

## Verification ladder for Go service changes

For a Go service change that touches storage + migration:

1. `gofmt -l <touched-pkgs> <cmd>/` — must be empty
2. `go build -mod=vendor <touched-pkgs>` — must exit 0
3. `go vet -mod=vendor <touched-pkgs>` — accept pre-existing warnings; flag only NEW warnings (compare against `origin/master`)
4. `go test -mod=vendor -count=1 <touched-pkgs>` — must pass
5. (Pre-push only) re-verify after rebase against current master, NOT just before commit
6. (Bonus) check `gh pr checks <num>` after push for the CI run

Each level is cheap and catches a different class of issue. **Skipping the post-rebase re-verify is the most common omission.**

## How to assess "is this PR comment certain enough to act on?"

Classify before acting:

| Category | Certainty | Action |
|---|---|---|
| Mechanical fix (gofmt, typo, dead import) | Always certain | Just do it |
| Comment / doc edit per reviewer's explicit request | Usually certain | Minimal change matching the request |
| Behavior change with reviewer's explicit ask | Certain only if you understand WHY and the change is local | Otherwise hold |
| Bot AI suggestion | Almost never certain | Treat as low signal unless it matches an explicit human reviewer request |
| Design pushback ("should we do X instead of Y?") | Never certain by yourself | Needs design-lead input |

## See also

- [[12 - Issues Encountered & Fixes]] for the concrete incidents these lessons came from
- [[00 - Index]] for the issue map
- Up: [[00 - Index]]

