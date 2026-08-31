---
title: Deep presentation and full clone of the setup
type: moc
created: 2026-08-30
---

# How to present this, and how to set it up so it actually works the same way

This is the long-form talk + clone guide. Short deck: [[Presentation outline]]. Click-path setup: [[How to set this up]].

**What “the same way” means:** not “they installed Obsidian.” It means a **new Cursor chat**, on a **new ticket**, without you pasting a 2,000-word bootstrap, still: audits before planning, uses a worktree, treats notes as a deliverable, does not invent GitHub state, classifies review comments, and writes a pattern when it learns something. That only happens if **Cursor rules + skills + vault** all exist. The plugin alone is not enough.

---

# Part 1 — How I would present it

Aim for a **60–75 minute workshop**, not a 10-slide pitch. The audience should feel the cost of a blank agent, then see the same agent with memory, then leave with a clone checklist.

## 1.1 Open with a story, not a tool (5 min)

Do **not** start with “Obsidian is a markdown wiki.” Start with a failure they recognize.

**Story (true to this repo):** On ENG-932537, a plan to add `products` on the v1 load path was drafted as ~9 steps (vendor models, handlers, filters). A two-grep audit showed `LoadObjectRequest.Products` already existed, the converter already copied it, and the filter allow-list already had `products`. The work collapsed to a couple of line edits. The lesson is not “grep is clever.” The lesson is **a blank agent will re-plan the universe every time unless that lesson is outside the chat.**

Second story: ENG-949861. The human brief said `authoringScope` is an **array** and “contains NC.” The code on ENG-915519 had already flipped it to a **scalar**. An agent that only has the Jira paste will implement the wrong check. An agent that is required to **read the definition before writing** implements `== "NC"`. That instruction (“VERIFY BEFORE YOU WRITE”) was given in August as an explicit operating rule. The vault’s job is to make that rule load **without** the paste.

**Line for the room:** Cursor is not underperforming because the model is weak. It is underperforming because **every session discards the last four months** unless we compile them.

## 1.2 Name the performance claim honestly (8 min)

Two different claims. Keep them on separate slides.

### Claim A — measured (WikiSkill paper)

Tang et al., [WikiSkill](https://arxiv.org/html/2608.27454): agents that **compile traces into a persistent wiki**, then **compile the wiki into short skills**, beat agents that only mutate a skill file from the last rollout.

- Qwen family average accuracy vs no-skill: **+12.3 / +17.5 / +23.9** points as models get larger.
- Persistent wiki for the skill proposer (Gemini Flash ablation): **48.7% → 63.7%**.
- Giving the **solver** the full wiki during the task **hurt** (63.7% → 60.9%). Inference should follow **skills**, not swallow the graph.
- Skills transfer across models; 9B + WikiSkill beat 27B with no skills on their average.

**Say out loud:** those numbers are math, search, spreadsheets, ALFWorld. They are **not** “our PRs merge 24% faster.”

### Claim B — operational (this team, May–August)

What we actually observed, without a fake KPI:

- **Less re-derivation.** Related tickets (authoringScope → mutation guard → products → shard copy → external identities) are a map, not tribal knowledge.
- **Fewer class-of-bug repeats.** Omitted SELECT lists, bootstrap dropping a field, Tavern ASCII, vendor `gofmt` cascades, `$fv` b3 vs b4, ETag retry-on-same-call — each became a **named pattern** after it burned us once.
- **Session continuity.** “Where I am,” decision log vs open questions, inverted-pyramid indexes — a new chat can read the index instead of the whole transcript.
- **Correctness under conflicting briefs.** User/Jira can be stale (array vs scalar). The durable rule is: **code wins; label VERIFIED vs INFERRED.**
- **New-agent ramp.** We used to paste a “starter pack” at the start of every chat. That paste **is** the skill layer. The setup’s job is to stop requiring the paste.

**Do not put a velocity percentage on this slide.** Pitch **reset cost** and **repeat-failure cost**.

## 1.3 Show the three layers on a whiteboard (8 min)

Draw three boxes. Walk a single ticket through them.

```
inbox/ENG-949861/00 - Index.md     RAW     human map, status, links
iam-themis/.notes/...CONTEXT.md   RAW     SHAs, commands, CI, decisions
wiki/entities/eng-949861.md       WIKI    stable facts + links
wiki/patterns/mutation-guard.md   WIKI    how we fail / how we work
~/.cursor/skills/iam-ticket-...   SKILL   what the agent RUNS this turn
```

**Punchlines:**

1. Graph view is the index. Query is walking wikilinks, not a chatbot with the whole vault in context.
2. `.notes/` never moves to Obsidian as the system of record for evidence.
3. Skills are short. If a skill is wrong, revert the skill; **keep the pattern.**

Live: open Graph, click authoringScope → ENG-949861 → AllowOperation. That is the demo of “related work in 15 seconds.”

## 1.4 Walk the four-month evolution (10 min)

This is the part most talks skip. It is why *your* setup is not “I installed a plugin.”

| When | What you trained the agent to do | Where it lived | What still broke |
| --- | --- | --- | --- |
| May 2026 | ENG-915519: inverted-pyramid indexes, decision log ≠ open questions, “where I am,” question-framing template, review-comment certainty, verification ladder | Vault ticket folders + `ntnx-api-iam/.notes/AUTHORING_SCOPE_CONTEXT.md` | Next chat still needed you to say “read the vault” |
| Jun | ENG-910347 / 910350: Themis-only tickets, worktrees per issue, don’t assume six repos | Same + `RESOLVER_SHARD_*` notes | Parallel agents switched branches under each other |
| May–Jul | ENG-932537: audit-before-plan, notes-as-deliverable **Cursor rule** (`iam-products-field.mdc` alwaysApply), subject-only commits, `commit-tree` for Co-authored-by, ASCII Tavern, no `gofmt -w` vendor | `.cursor/rules/` + `.notes/PRODUCTS_FIELD_CONTEXT.md` | Rule was **ticket-specific**. Other tickets did not inherit the *process*, only products constraints |
| Jul–Aug | ENG-948242, ETag/migrations, deployment image skew, credential redaction, Gerrit `refs/for` vs GitHub PR | Technical stack notebook | Bootstrap prompt grew into a novel |
| Aug 3 | Explicit **starter pack** / master bootstrap: implement don’t analyze-only; recon → implement → validate → integrate → document; classify CI (code vs fixture vs infra vs merge); Issue Memory before coding | Pasted into each new chat | If you forgot the paste, the agent was a May agent again |
| Aug 10 | “Operate like a careful senior engineer”: VERIFY BEFORE YOU WRITE; reuse don’t reinvent; scope tightly; prove it; leave a trail | Pasted operating system | Still paste-dependent |
| Aug 20–24 | “Inherit lessons from all prior issues”; Issue Memory schema; worktree same as other issues; test E2E before push | Prompt add-on | Memory still reconstructed from grep + hope |
| Aug 30 | Obsidian MCP + Karpathy wiki + WikiSkill loop + skills compiled **from** those lessons | Vault `wiki/patterns/` + `~/.cursor/skills/` + `wikiskill-loop.mdc` | **This** is the compile step the paper describes |

**Say:** The plugin did not create the intelligence. **You** created it over four months of tickets. The vault is the compiler output. Cursor skills are the executable object file. WikiSkill is the name for that compile.

## 1.5 Live agent loop (8 min)

New Cursor chat, project rooted on `nutanix-core` (or a demo clone):

> Starting ENG-…. Search the vault for related notes. Do not dump the whole wiki. Audit the code for what is already wired. Five-line plan. Label VERIFIED vs INFERRED. Do not invent GitHub state.

Then, after a canned “we learned X”:

> Add `wiki/patterns/<slug>.md` with failure mode, workaround, evidence. Append `wiki/log.md`. Update only the inbox index status line.

Show that Settings → Rules lists `wikiskill-loop` and `iam-products-field`. If the rule is missing, the live demo **fails on purpose** — that is the teaching moment: vault without rules is a museum.

## 1.6 Close (3 min)

Three sentences:

1. Performance here is **compounding procedural knowledge**, which the paper measured, plus **our** ticket graph, which we have not A/B tested as a percentage.
2. Clone the **loop**, not the ENG-* files.
3. If you only install the plugin, you get pretty Graph view. If you add always-on rules + skills + end-of-turn patterns, you get a teammate that does not forget June.

---

# Part 2 — The personal instruction corpus (what to put in skills/rules)

These are the instructions you have actually been giving. A clone that omits them will not “work the same way.” Map each to **rule** (always on) vs **skill** (when IAM ticket) vs **wiki pattern** (knowledge).

## 2.1 Senior-engineer operating system (Aug 10 — keep as skill + bits in always-on rule)

From the “Operate like a careful senior engineer” prompt:

- **VERIFY BEFORE YOU WRITE.** Never call a function, constant, type, or column without reading the definition. Do not infer APIs.
- **REUSE, DON’T REINVENT.** Mirror an existing pattern in this repo, a sibling, or a referenced PR. If the helper only exists on an unmerged branch, reimplement the minimal local equivalent and say so.
- **SCOPE TIGHTLY.** Only what the task requires. No drive-by refactors. Ask before extra work.
- **TRACE THE WHY.** Tie non-obvious decisions to a requirement, review comment, code, or verified constraint. Surface conflicts. Push back with evidence.
- **PROVE IT WORKS.** Build + vet/lint + relevant tests. Separate errors you introduced from pre-existing warnings.
- **STATE HYGIENE.** Expected branch/worktree before commit. Clean author, no unwanted trailers. Fast-forward preferred. `--force-with-lease` only with verified remote tip. Never push shared/protected or force-push without approval.
- **LEAVE A TRAIL.** Notes/decision log every substantive turn.
- **COMMUNICATE.** Lead with outcome. Don’t claim done until verified.

## 2.2 Release-agent bootstrap (Aug 3 — IAM skill)

- Don’t stop at analysis; implement, test, and push **when asked**.
- Minimal, scoped, reversible. Root cause over symptom patches.
- Notes/docs are **deliverables**, not optional.
- Workflow: **Recon** (repos, open PR mergeability/CI/behind) → **Implement** → **Validate** (focused tests; YAML/Tavern syntax) → **Integrate** (merge base, revendor with **intent**) → **Document**.
- Classify the task: code bug vs test fixture drift vs infra/env vs merge/dependency drift.
- CI: isolate feature code vs pipeline inputs (wrong image, `$fv`, TLS).
- No speculative refactors. State VERIFIED vs INFERRED. Flag credential leaks. If blocked, 1–2 concrete unblock paths.
- Final report: issue → cause → fix → evidence → next actions.

## 2.3 Issue Memory (Aug — wiki maintainer + start-of-ticket skill)

Before coding, for each relevant prior issue/PR:

- ID
- Root-cause class (code, test drift, dependency drift, infra/env, release/process)
- Exact fix pattern
- Validation commands that proved it
- Common regression signals

Reuse proven fixes first. Confidence labels: VERIFIED vs INFERRED.

This is exactly WikiSkill’s **pattern catalog**. Your `wiki/patterns/` is Issue Memory with filenames.

## 2.4 Process lessons compiled from tickets (May–Jul)

Already in vault process-lesson notes; must live in **skills** as procedures:

- One canonical `.notes/*_CONTEXT.md` per initiative; numbered sections; `> Correction note:` when the doc was wrong.
- Inverted pyramid on indexes (status first).
- Decision log separate from open questions.
- “Where I am” at session end.
- Question-framing template (headline + ID, background, A vs B, sub-questions, out of scope).
- `git rev-parse --abbrev-ref HEAD` before significant git (parallel agents).
- Sticky `go.mod` replace → vendor contamination; reset+action+verify in one shell or use worktrees.
- `--force-with-lease` not `--force`.
- Verification ladder: gofmt → build → vet (new only) → test → **again after rebase**.
- Review comments: mechanical / explicit human / behavior / bot / design-lead.
- Audit before planning (grep vendor, converters, column lists, filters).
- Vendor TEMP HACK + same-day upstream PR; prefer canonical style over markers if gofmt cascades.
- `git commit-tree` to strip Cursor Co-authored-by **without** `--no-verify`.
- Subject-only commits; bodies in `.notes/` and PR template.
- ASCII-only Tavern YAML.
- Bulk-edit search keys must include the **method name**.
- Verify pre-existing failures on bare HEAD (stash including untracked).

## 2.5 Technical-stack standing orders (Jul–Aug notebook)

- Re-verify GitHub/code/CI before acting; vault goes stale.
- No credentials in the vault or PR paste.
- Public field ownership chain: api-iam → iam-utils → themis → bootstrap → user-authn → deployment. Do not assume all six.
- E2E checklist: schema, conversion, storage/JSON, SQL + **every SELECT list**, sqlite/sqlmock, bootstrap, deploy, unit/tavern, vendoring/`$fv`.
- Troubleshoot by class: empty reads → column lists; seed but no rows → bootstrap converters; Tavern 428 → first failed stage; `$fv` mismatch → image pair; ETag mismatch → don’t retry same call.
- Reply “done” on review only if implemented.
- GitHub IAM = normal PR; Gerrit = `refs/for/<branch>`.
- Feature-specific decisions are **not** universal (products ≠ APs; authoringScope is scalar; etc.). State which lessons apply.

## 2.6 Ticket-shaped constraints (alwaysApply **file** rules, not the wiki)

- ENG-932537: Roles + Entities only; APs out of scope; `products` / `ProductList` / `text[]`; read-only API; log `PRODUCTS_FIELD_CONTEXT.md` every turn.
- ENG-915519: scalar `{NC, PC}`, server-stamped, immutable, omit-on-empty.
- Mutation guard: handler layer so Lattice ApplyChange still mutates; PC + NC-authored → 403.
- Per-ticket **worktree**, same as previous issues; E2E tests before push.

## 2.7 User-global Cursor rules (all projects)

- Commit only when asked; no `--no-verify`; no reckless force-push; HEREDOC commit messages.
- PRs via `gh` with summary + test plan.
- Web UI: verify in the browser.
- Gerrit: `refs/for`, never direct heads.

If a clone only copies Obsidian and skips 2.1–2.7, they will not get your agent.

---

# Part 3 — Exhaustive setup so it works the same way

Do this **in order**. Times assume a clean Mac with Homebrew.

## Phase A — Vault (Obsidian)

1. Install Obsidian. **Create one vault.** Open that folder. (Nested `.obsidian` was our first failure mode.)
2. Community plugins → Karpathy LLM Wiki (`karpathywiki`) → enable → reload.
3. Create:

```
Home.md
inbox/README.md
inbox/<TICKET>/00 - Index.md     # first real or demo ticket
wiki/entities/
wiki/concepts/
wiki/sources/
wiki/schema/config.md            # their domain vocabulary
wiki/patterns/                   # start empty or copy pattern *shape*
wiki/index.md
wiki/log.md
wiki/skill-impact.md
```

4. Settings → Karpathy LLM Wiki: wiki folder `wiki`, Schema **on**. Restart Obsidian.
5. Optional: LLM provider + Test Connection. Without it, Cursor writes wiki pages (Path B).

**Index template** (copy forever):

- Frontmatter: tags, status, updated
- One-line what/why
- Status at a glance (PRs, blockers)
- Read order of sibling notes
- Link to `repo/.notes/FOO_CONTEXT.md`
- Related tickets as wikilinks
- Not: SHAs (those go in `.notes/`)

## Phase B — Git repo memory

1. For each long initiative: `iam-themis/.notes/<INITIATIVE>_CONTEXT.md` (or the owning repo).
2. Sections: overview, decisions (D1…), files touched, gate evidence, PR body drafts, changelog with `YYYY-MM-DD` + `agent session N`.
3. Cursor project rule with `alwaysApply: true` **for that initiative’s constraints** (clone of `iam-products-field.mdc` style): logging discipline is mandatory.

Without `.notes/`, the vault becomes a fanfic of git.

## Phase C — Cursor talks to the vault

1. `brew install node` so `npx` exists.
2. `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "obsidian": {
      "command": "npx",
      "args": ["-y", "obsidian-mcp", "/ABS/PATH/TO/THEIR/VAULT"]
    }
  }
}
```

3. Restart Cursor → Settings → MCP → enable → trust.
4. Verify: agent can list vaults and read `Home.md`.

## Phase D — Compile instructions into skills (this is the 4-month payload)

Create **personal** skills (all workspaces):

`~/.cursor/skills/wikiskill-loop/SKILL.md`  
Three layers; inference vs maintain vs propose; vault path.

`~/.cursor/skills/iam-ticket-workflow/SKILL.md`  
Paste the **procedures** from Part 2.4–2.5 (audit, ladder, tavern, worktree, VERIFIED/INFERRED, Issue Memory). Keep under ~150 lines; put long recipes in `PURPOSE.md` or `reference.md`.

`~/.cursor/skills/iam-ticket-workflow/PURPOSE.md`  
Links to `wiki/patterns/*`.

**Do not** put the entire technical-stack notebook in the skill. That is wiki. The skill **points** at it.

Omit `disable-model-invocation` so Cursor can auto-trigger from ENG-*, IAM, tavern, lattice, etc.

## Phase E — Always-on rules (the thing that replaces the paste)

1. User: `~/.cursor/rules/wikiskill-loop.mdc` with `alwaysApply: true` (WikiSkill loop + “do not dump wiki” + “do not invent SHAs”).
2. **Also copy into** `<monorepo>/.cursor/rules/wikiskill-loop.mdc` so teammates and cloud agents get it.
3. Keep ticket rules (`iam-products-field.mdc`, tavern k3d) as separate files.
4. Confirm **Settings → Rules** shows them. If user-level `.mdc` does not appear, the **repo** copy is the one that matters for “same way at work.”

Optional: Cursor Settings → User rules: paste the **VERIFY BEFORE YOU WRITE** block (2.1) so it is in the system prompt even when skills fail to attach.

## Phase F — First ingest and first pattern

Path A: `Karpathy LLM Wiki: Ingest from folder` → `inbox`.  
Path B: Cursor prompt to create source/entity/concept pages from the index, bidirectional links, no invented SHAs.

Then write **one** pattern from a real scar (e.g. audit-before-planning). Append `wiki/skill-impact.md` when you add the skills.

## Phase G — Prove a new chat works

Open a **new** agent in the repo. Do **not** paste the bootstrap. Ask:

> What do you do before planning an IAM ticket?

Pass if it says: vault search, Issue Memory/patterns, audit code, worktree, VERIFIED vs INFERRED, `.notes/` at the end.  
Fail if it starts coding or asks you to paste context.

That gate **is** the demo of “same way.”

---

# Part 4 — How the agent evolved with you (narrative for the talk)

Speak in first person as the engineer, or third person as “the agent.” Either way, keep it chronological and specific.

**May.** We started ENG-915519 with a vault of numbered notes so a 12-file feature had a read order. We learned to split decisions from open questions because mixing them made every session re-litigate scalar vs array. We wrote a question-framing template after Glean/colleague dumps failed. We learned review bots are not requirements. The agent still forgot this the next Monday.

**June.** Shard copy and external-identity tickets taught “not every field needs ntnx-api-iam.” Worktrees became mandatory after two agents shared a checkout and `HEAD` lied. Themis-only scope is now a pattern, not a guess.

**July.** Products taught audit-before-plan the hard way. We promoted notes-as-deliverable to an **alwaysApply rule** because asking “did you update the notes?” meant we had already lost the turn. Commit-msg hooks vs subject-only convention produced the `commit-tree` recipe. Tavern Python 2.7 produced ASCII-only. Vendor gofmt produced “never format generated files.” Search-metadata and deployment work taught `$fv` image pairing and “don’t put Artifactory passwords in notes.”

**August.** You stopped relying on my memory and started **pasting an operating system**: senior-engineer rules, IAM bootstrap, Issue Memory. ENG-949861 proved verify-before-write: the Jira-shaped brief said array; the branch said scalar; the implementation followed the branch and documented the correction. You asked for the same worktree ritual “as the other issues” and E2E before push — that is now in the skill. Then you installed Obsidian MCP and Karpathy wiki because **paste does not scale**. WikiSkill named what we were already doing: traces → wiki → skills, wiki never rolls back.

**The honest arc:** intelligence did not appear when we enabled a plugin. Intelligence was **logged**, then **ruled**, then **pasted**, then **compiled**. Anyone cloning only step “install plugin” is in May 2026.

---

# Part 5 — Suggested run of show (75 min)

| Min | Block | You show |
| --- | --- | --- |
| 0–5 | Story: 9-step plan vs 2 greps; array vs scalar | Two sentences each |
| 5–13 | Two claims: paper numbers vs operational | Chart + “not IAM KPI” |
| 13–22 | Three layers + Graph walk | Live Obsidian |
| 22–35 | Four-month table | This note’s timeline |
| 35–50 | Setup phases A–E on a **clean** vault or a clone laptop | [[How to set this up]] plus Phase D–E |
| 50–60 | New chat gate (Phase G) | Pass/fail |
| 60–70 | Live pattern write | wiki/patterns + log |
| 70–75 | Q&A traps | Nested vault, no npx, @-wiki, Query Wiki ≠ Cursor, copying ENG-* |

Handout: this file + [[How to set this up]] + zip of `~/.cursor/skills/{wikiskill-loop,iam-ticket-workflow}` + the two `.mdc` rules **with paths stripped**.
