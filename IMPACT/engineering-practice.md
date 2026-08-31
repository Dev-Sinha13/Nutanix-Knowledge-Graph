---
title: IMPACT — engineering practice as a deliverable
updated: 2026-08-31
---

# Engineering practice — compounding impact on the IAM program

Back: [IMPACT index](README.md)

This page is **not** a ticket. It is how the program stayed coherent across four months, six repos, and parallel agents. Nutanix paid for Global IAM once; this layer is why ticket three was cheaper than ticket one.

---

## 1. The problem

Without a durable map outside chat:

- Agents re-plan work that is already wired (932537 v1 load path).
- Briefs disagree with branches (`authoringScope` array vs scalar).
- CI turns red for punctuation (Tavern Python 2.7) or mixed-version rigs and looks like “IAM is broken.”
- SHAs, PR bodies, and “why we reverted AP products” die when the thread resets.

---

## 2. How it was solved

Three layers that must stay separate:

| Layer | Where | Role |
| --- | --- | --- |
| Raw | `inbox/` + repo `.notes/` | Evidence, SHAs, CI |
| Wiki | `wiki/` | Concepts, related work, patterns |
| Skills | Cursor `SKILL.md` + always-on rules | Procedures to run **this turn** |

Inference: skills + a few pages. Do not dump the wiki. After a real turn: update notes, patterns, log.

---

## 3. Impact

Fewer repeated design mistakes, faster ticket start via related ENG-* links, new chats that do not require a bootstrap paste, ticket contracts (products-field rule) still apply on top.

---

## 4. How it was done

May: indexes, decision log ≠ open questions, verification ladder.  
Jun: Themis-only scope, worktrees, HEAD check.  
May–Jul: audit-before-plan, notes-as-deliverable, commit-tree, ASCII tavern.  
Jul–Aug: image skew, credential redaction, Gerrit `refs/for`.  
Aug 30: Obsidian graph + MCP + skills so paste is no longer the OS.

---

## 5. Technologies and skills

Obsidian (wikilinks, Graph), Cursor rules/skills/MCP, git worktrees, `gh`, Gerrit `refs/for`, Go toolchain, Tavern/CI literacy.

---

## 6. Significant technical difficulties

The significant ones are **encoded as patterns**, not as a list of debugger sessions: audit-before-planning, verify-pre-existing-on-bare-head, vendor-gofmt-cascade, ascii-only-tavern, branch-drift-across-shells, question-framing, review-comment-certainty. Each ticket page §6 points at the ones that actually changed the design.

See [How to set this up.md](../How%20to%20set%20this%20up.md) and [cursor/setup.md](../cursor/setup.md).
