---
title: Presentation — compounding wiki for Cursor
type: moc
created: 2026-08-30
---

# Presentation outline (speaker notes)

Open the live deck beside chat: ask Cursor for the **Obsidian WikiSkill talk** canvas, or follow these slides in order. Full setup script: [[How to set this up]].

**Claim in one sentence:** We compile experience into a wiki, compile the wiki into short Cursor skills, and we do **not** dump the wiki into every prompt.

**Cite:** Tang et al., *WikiSkill*, [arXiv:2608.27454](https://arxiv.org/html/2608.27454). Numbers below are their averages across five benchmarks, not IAM PR metrics. Say that out loud.

---

## Slide 1 — Title (1 min)

**Title:** A compounding engineering wiki for Cursor agents

**Say:** New chats used to start at zero. This setup is how a teammate (or a new agent) gets the same map, the same procedures, and the same failure patterns.

**On screen:** 3 layers · paper gains on Qwen family · one alwaysApply rule.

---

## Slide 2 — Problem (3 min)

Without this:

- Lessons die in old threads
- Notes go stale; models invent SHAs
- A vault can exist and still be ignored
- “@ the whole vault” feels smart and is the wrong ablation

With this: graph for related work, `.notes/` for evidence, skills for how to work, patterns so we do not repeat the same vendor/tavern/gofmt failure.

---

## Slide 3 — Architecture (4 min)

| Layer | Where | Role |
| --- | --- | --- |
| Raw | `inbox/` + repo `.notes/` | Immutable sources, SHAs, CI |
| Wiki | `wiki/` entities, concepts, **patterns** | Compounding knowledge, never rolled back |
| Skills | `~/.cursor/skills/` | Short procedures the agent **runs**; can be reverted |

**Punchline:** Solver uses skills + a few pages. Wiki is updated **after** the turn. Paper: giving the inference agent the full wiki during the task **hurt** skill quality (Gemini Flash ablation 63.7% → 60.9%).

---

## Slide 4 — Performance evidence (5 min)

WikiSkill vs no-skill, average accuracy (%), Table 1:

| Model | No skill | WikiSkill | Delta |
| --- | --- | --- | --- |
| Qwen-3.5-4B | 26.2 | 38.5 | +12.3 |
| Qwen-3.5-9B | 29.9 | 47.4 | +17.5 |
| Qwen-3.6-27B | 39.4 | 63.3 | +23.9 |
| Gemma-4-31B | 41.3 | 54.9 | +13.6 |
| Gemini-3.5-Flash | 49.5 | 68.1 | +18.6 |

Also: 9B + WikiSkill (47.4) beat 27B with no skills (39.4). Persistent wiki for the skill proposer: 48.7 → 63.7 avg.

**Say:** Gains grow with model scale. Skills transfer across models. We are applying the **same split of knowledge vs procedure**, not claiming +24 points on iam-themis.

---

## Slide 5 — What we expect on our work (3 min)

- Faster ticket start (related ENG-* already linked)
- Fewer repeated process failures (patterns)
- New Cursor session without a human dump
- Ticket contracts (e.g. products-field rule) still apply on top

Do not put a fake “+40% PR velocity” number on the slide.

---

## Slide 6 — What they copy vs not (2 min)

**Copy:** folder contract, plugin, schema *shape*, MCP snippet, two skills, alwaysApply rule, one blank ticket template.

**Do not copy:** ENG-* bodies, keys, nested vault, “Query Wiki uses Cursor.”

---

## Slide 7 — Setup (4 min, point at [[How to set this up]])

1. One vault, actually open  
2. Karpathy LLM Wiki enabled  
3. `inbox/` + `wiki/{entities,concepts,sources,schema,patterns}` + `Home.md`  
4. Schema on, wiki folder `wiki`, restart  
5. First `00 - Index.md`  
6. Node + `~/.cursor/mcp.json` + MCP on  
7. Ingest (plugin **or** Cursor) → Graph view  
8. Skills + `wikiskill-loop.mdc` alwaysApply (user **and** repo `.cursor/rules/`)

Plugin LLM key optional.

---

## Slide 8 — Exact Cursor artifacts (3 min)

**Rule** `wikiskill-loop.mdc` (`alwaysApply: true`):

- Inference: skills, not the whole wiki  
- End of real turn: inbox index, `.notes/`, `wiki/patterns/`, `wiki/skill-impact.md` if a skill changed  
- Wiki never rolls back  

**Skills**

- `~/.cursor/skills/wikiskill-loop/SKILL.md`  
- `~/.cursor/skills/iam-ticket-workflow/SKILL.md` (rename for other domains) + `PURPOSE.md` → patterns  

Confirm Settings → Rules / Skills are enabled. Repo copy so clones get the rule.

Existing `iam-products-field.mdc` stays — that is a ticket contract, not the vault loop.

---

## Slide 9 — Live loop (2 min)

Fresh chat:

> Starting ENG-…. Search the vault for related notes. Do not dump the whole wiki. Audit what is already wired. Five-line plan. Do not invent GitHub state.

Then:

> Add `wiki/patterns/<slug>.md` and append `wiki/log.md`.

Show Graph view.

---

## Slide 10 — Close / Q&A (2 min)

Three takeaways: compounding memory · wiki for maintainer not solver · setup is folders + MCP + two skills + one rule.

Traps: nested vault, wrong window, no npx, two different LLMs, @-entire-vault.

---

## Timing

| Block | Minutes |
| --- | --- |
| Problem + architecture | 8 |
| Paper + honest caveat | 8 |
| Setup + rules | 8 |
| Live loop | 4 |
| Q&A | 7 |
| **Total** | **~35** |

Stretch to 50 with the full [[How to set this up]] walkthrough on a clean demo vault.
