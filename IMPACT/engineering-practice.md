---
title: IMPACT — engineering practice as a deliverable
---

# Engineering practice — compounding impact on the IAM program

**Company impact:** The tickets above are the product. This page is **why the program did not fall apart across four months, six repos, and parallel agents.** Nutanix paid for Global IAM once. The notes, rules, and skills are how the second and third tickets were cheaper than the first — and how a new engineer (or a new Cursor chat) does not re-break scalar vs array, ASCII tavern, or vendor gofmt.

Back: [IMPACT index](README.md)

---

## What would have happened without this layer

- **Re-planning already-wired code.** ENG-932537’s v1 load path was a 9-step plan until two greps showed it was done. Blank agents will re-plan that universe every Monday.
- **Wrong field shape.** ENG-949861’s Jira-shaped brief said `authoringScope` is an array. The branch said scalar. Verify-before-write is the only reason the guard is `== "NC"`.
- **CI red for punctuation.** Python 2.7 Tavern + one em-dash = collector crash before any IAM assertion. That looks like “IAM is broken” to a dashboard.
- **Vendor diffs nobody can review.** `gofmt -w` on mixed-indent generated files.
- **False green upgrades.** Empty tenant list as completion; ETag retry on a stale expected value; in-memory flag treated as fleet completion.
- **Lost PR bodies and SHAs** on chat reset — `.notes/` became part of the **definition of done**, not an afterthought (`iam-products-field.mdc` logging discipline).

Those are **company** costs: review time, CI minutes, support tickets on wrong UUIDs, NC UI showing empty products.

---

## Timeline (May–August 2026)

| When | What got encoded | Where it lives |
| --- | --- | --- |
| May | Inverted-pyramid indexes; decision log ≠ open questions; question-framing; review-comment certainty; verification ladder | ENG-915519 inbox + `AUTHORING_SCOPE_CONTEXT.md` |
| Jun | Themis-only scope; worktrees; HEAD check before git | ENG-910347 / 910350 |
| May–Jul | Audit-before-plan; notes-as-deliverable rule; subject-only commits; `commit-tree` vs `--no-verify`; ASCII tavern; no vendor gofmt | ENG-932537 + `.cursor/rules/iam-products-field.mdc` |
| Jul–Aug | Entity search, image skew, credential redaction, Gerrit `refs/for` vs GitHub | Technical stack; ENG-948242 |
| Aug 30 | Vault graph + Obsidian MCP + skills so the **paste is no longer the operating system** | `wiki/`, `cursor/`, `~/.cursor/skills/` |

---

## Concrete practices that changed outcomes

1. **Two stores.** Vault = navigation and concepts. Repo `.notes/` = SHAs, CI, commands. Duplicating SHAs onto concept pages is how notes go stale.
2. **Handler vs storage.** Mutation and identity validators on the API path; Lattice must still write. 949861 and 910350 both depend on this.
3. **Do not assume six repos.** 910347/910350 are Themis-only. Opening ntnx-api-iam on those tickets is wasted time.
4. **Worktrees.** Parallel agents switching `HEAD` under each other was a real incident class (`git rev-parse --abbrev-ref HEAD` before significant git).
5. **Classification of review comments.** Mechanical: do. Design: ask Manish. Bot: ignore unless a human asked. Saved implementing fake requirements.
6. **Gerrit vs GitHub.** IAM GitHub repos: normal PRs. AOS/main-style: `refs/for/<branch>`, never `refs/heads`.
7. **Secrets.** Verbose deployment logs printed an Artifactory credential on 948242 — rotate, redact pipelines, never paste into the vault.

---

## How this vault is itself a deliverable

The knowledge graph is not a slide. It is:

- Related-work edges (who vs what, ApplyChange vs shard copy, field vs guard).
- Patterns under `wiki/patterns/` (audit-before-planning, ASCII tavern, vendor gofmt, …).
- Cursor skills that **run** those procedures without pasting a novel each chat.

WikiSkill split used here: **inference uses skills + a few pages**, not the whole wiki. Dumping the vault into every prompt is the ablation that **hurts**.

For clone / talk setup see [How to set this up.md](../How%20to%20set%20this%20up.md) and [cursor/setup.md](../cursor/setup.md).
