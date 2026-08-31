---
title: Setup demo — Obsidian + Cursor + WikiSkill
type: moc
created: 2026-08-30
---

# Setup demo (step by step)

Use this as the live script. One vault. Do **not** create a vault inside another vault.

**This machine’s live Cursor files** (rules, skills, MCP) are copied into [[cursor/00 - Index]] so you can edit them in Obsidian. Operational checklist: [[cursor/setup]].

Replace `~/Documents/WorkVault` with the attendee’s path. On this machine the working vault is:

`/Users/dev.sinha/Documents/Obsidian Vault/Nutanix-Core`

---

## 0. What you will have at the end (~25 min setup + 10 min first note)

- Obsidian vault with `inbox/` (sources) and `wiki/` (graph)
- Karpathy LLM Wiki plugin enabled
- Cursor talking to the vault over MCP
- WikiSkill loop: raw notes ≠ wiki ≠ Cursor skills
- One sample ticket ingested into the graph

---

## 1. Install Obsidian and create one vault

1. Install Obsidian from https://obsidian.md (macOS: `Obsidian.app`).
2. Open Obsidian → **Create new vault**.
3. Name it something like `WorkVault`.
4. Put it at `~/Documents/WorkVault` (or similar). Click **Create**.
5. Confirm you opened **that** folder, not a parent that happens to contain it.

**Say:** Community plugins and Karpathy Wiki are per-vault. If the plugin “isn’t there,” you are in the wrong vault.

---

## 2. Enable community plugins and install Karpathy LLM Wiki

1. Settings → **Community plugins** → turn off Restricted mode if prompted.
2. **Browse** → search `Karpathy LLM Wiki` (id `karpathywiki`).
3. Install → **Enable**.
4. Reload Obsidian (command palette: `Reload app without saving`) if the welcome note does not appear.

You should see a note under `wiki/` such as `Welcome to Karpathy LLM Wiki`.

**Say:** First-run welcome lives in `wiki/`. If LLM is not configured it says so in frontmatter (`llm_config_status: failed`). That is expected until step 8.

---

## 3. Create the folder contract

In the file explorer, create:

```
inbox/
wiki/
  entities/
  concepts/
  sources/
  schema/
  patterns/
```

Add two notes:

**`Home.md`** (vault root) — dashboard: what inbox vs wiki is, and “start here.”

**`inbox/README.md`** — “Source notes for ingest. Do not put generated wiki pages here.”

**Say:** `inbox/` is human / agent source. `wiki/` is compounding knowledge. Never overwrite inbox with generated pages. Repo `.notes/` files still own SHAs and CI logs.

---

## 4. Seed the wiki schema (domain vocabulary)

Create `wiki/schema/config.md` (Karpathy uses this when Schema is on).

Minimum to paste and then customize:

- Tickets (`ENG-*` or your prefix) → entity / project
- Products (your NC/PC equivalents) → entity / product
- Repos → entity / other
- Fields and mechanisms → concept / term or method
- Keep rival products as **separate** entities
- `wiki/patterns/` = failure modes and strategies, not facts

Enable Schema: **Settings → Karpathy LLM Wiki → Schema → Enable**.

Set **Wiki folder** to `wiki`. Restart Obsidian after changing the wiki folder.

---

## 5. Write the first source note (before any ingest)

Create `inbox/DEMO-1/00 - Index.md`:

```markdown
---
tags: [demo, notes]
status: active
---

# DEMO-1 — <one-line problem>

> One sentence: what this is and why it exists.

## Status
- Not started.

## Related
- (link sibling tickets once they exist)

## Canonical evidence
- Repo notes file: `<repo>/.notes/DEMO_CONTEXT.md`
```

**Say:** This is the raw layer. Ingest copies *from* here *into* `wiki/sources/` plus entity/concept pages. The index stays the human map.

---

## 6. Install Node (required for Cursor MCP)

In a normal terminal (not sandboxed):

```bash
node --version   # need a real Node, not only Cursor’s bundled binary
# if missing:
brew install node
```

`npx` must be on `PATH` (`/opt/homebrew/bin/npx` on Apple Silicon).

---

## 7. Point Cursor at the vault (MCP)

Create or edit `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "obsidian": {
      "command": "npx",
      "args": [
        "-y",
        "obsidian-mcp",
        "/FULL/PATH/TO/WorkVault"
      ]
    }
  }
}
```

Use an **absolute** path. Spaces in the path are fine inside that JSON string.

Then:

1. Restart Cursor (or reload windows).
2. **Settings → MCP & Integrations** (or Tools & MCP).
3. Toggle **obsidian** on. Trust / approve the first run.
4. First start downloads `obsidian-mcp` via `npx -y` (needs network).

**Verify:** ask the agent “list my Obsidian vaults” or confirm tools like `obsidian_list_vaults` / `read-note` appear.

**Say:** MCP lets Cursor read/write notes. It is not the Karpathy Query Wiki chat bubble. Those are two different LLMs.

---

## 8. Optional: plugin LLM (Query Wiki in Obsidian)

Only if you want the Obsidian chat ribbon:

1. **Settings → Karpathy LLM Wiki → LLM Provider**.
2. Pick provider, paste API key, pick model.
3. **Test Connection** must show success.
4. **Save Settings**.
5. Command palette: `Karpathy LLM Wiki: Recreate Wiki Welcome Note`.

Without this key, skip to step 9 and use Cursor to build wiki pages.

---

## 9. First ingest

**Path A — plugin (needs step 8)**

1. Command palette → `Karpathy LLM Wiki: Ingest from folder`
2. Choose `inbox`
3. Wait (tens of seconds per note)
4. Open Graph view; you should see new `wiki/entities/` and `wiki/concepts/` files

**Path B — Cursor (no plugin key)**

Prompt:

> Read `inbox/DEMO-1/00 - Index.md`. Create wiki source, entity, and concept pages under `wiki/` following `wiki/schema/config.md`. Bidirectional wikilinks. Do not invent SHAs or PR numbers.

Then open **Graph view**.

**Say:** Path B is how this vault was first populated. Plugin ingest is nicer when the key works; Cursor is the fallback.

---

## 10. WikiSkill layers (this is the teaching punchline)

Draw this on the board or point at `Home.md`:

| Layer | Where | Role |
| --- | --- | --- |
| Raw | `inbox/`, git `.notes/` | Immutable evidence |
| Wiki | `wiki/` (entities, concepts, sources, **patterns**) | Compounding knowledge; never roll back |
| Skills | `~/.cursor/skills/` | Short procedures the agent **runs** |

Create `wiki/patterns/` if missing. Add one pattern from a real failure, e.g. `wiki/patterns/audit-before-planning.md`:

- Failure mode
- Workaround
- Evidence (link the source note)

Create `wiki/skill-impact.md` (table: date, skill, change, accepted/rejected, why).

**Say:** Ablation in WikiSkill: if the solver reads the **whole wiki** while doing the task, skill quality drops. Inference follows **skills** plus a few pages. Session end updates the wiki.

---

## 11. Cursor skills + always-on rule

**Skills** (personal, all projects): `~/.cursor/skills/<name>/SKILL.md`

Minimum two:

1. `wikiskill-loop` — three layers; maintain wiki at end of a real turn
2. `iam-ticket-workflow` (or rename to their domain) — audit before plan, verification ladder, don’t invent SHAs

Each skill: YAML `name` + `description` (what **and** when). Optional `PURPOSE.md` linking to `wiki/patterns/`.

**Rule** so a **new** agent actually does this:

- User: `~/.cursor/rules/wikiskill-loop.mdc` with `alwaysApply: true`
- And/or copy into the git repo: `<repo>/.cursor/rules/wikiskill-loop.mdc`

Confirm in **Cursor Settings → Rules** and **Skills** that they are enabled.

**Say:** Without the rule, the vault exists and agents ignore it. That was the gap before WikiSkill.

---

## 12. First “real” loop (2 minutes on stage)

Prompt a **new** Cursor chat in the project:

> We are starting DEMO-1. Search the vault for related notes. Do not dump the whole wiki. Audit the code for anything already wired. Then propose a 5-line plan. Do not invent GitHub state.

After you accept a fake “lesson”:

> Add a pattern under wiki/patterns/ and append wiki/log.md. Do not change inbox except the index status line.

Show Graph view again: new pattern linked from the index.

---

## 13. Daily use (what they copy after the demo)

1. New ticket → `inbox/<ID>/00 - Index.md`
2. Cursor: search vault → audit code → implement
3. Evidence → repo `.notes/`
4. Map/status → inbox index
5. Durable failure/strategy → `wiki/patterns/`
6. Promote stable facts → entity/concept pages
7. Change a Cursor skill only from patterns + this turn; log in `wiki/skill-impact.md`

Re-verify GitHub/code before acting. Vault notes go stale.

---

## Pitfalls to call out live

- **Nested vault:** `.obsidian` inside another `.obsidian` — plugin settings won’t match the window you think you’re in.
- **Wrong vault open:** parent `Obsidian Vault` vs `Nutanix-Core`.
- **No Node / no npx:** MCP stays red.
- **Query Wiki vs Cursor:** two different LLMs; one key does not power the other.
- **@-ing the entire vault:** opposite of WikiSkill; don’t.

---

## Copy-paste checklist (attendee)

- [ ] One vault created and actually open
- [ ] Karpathy LLM Wiki installed and enabled
- [ ] `inbox/` + `wiki/{entities,concepts,sources,schema,patterns}`
- [ ] `Home.md` + first `inbox/.../00 - Index.md`
- [ ] Node + `~/.cursor/mcp.json` + MCP toggle on
- [ ] Schema on, wiki folder `wiki`
- [ ] First ingest (plugin **or** Cursor)
- [ ] Graph view shows links
- [ ] `wikiskill-loop` skill + always-on rule enabled
- [ ] One pattern note written
