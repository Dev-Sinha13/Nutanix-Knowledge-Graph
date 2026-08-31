---
title: Why a knowledge graph, and why the vault is structured this way
type: moc
created: 2026-08-30
---

# Why a knowledge graph — and the reasoning behind every setup choice

This note is the “why,” not the click path. Clicks: [[How to set this up]]. Four-month instructions: [[Deep presentation and full clone]].

---

## 1. What problem a knowledge graph is for

IAM work here is not one file and one PR. A typical Global IAM change has:

- Several **tickets** that look independent in Jira and are not (authoringScope without a mutation guard is an incomplete story).
- Several **repos** with a directed ownership chain (schema → utils → themis → bootstrap → deploy).
- Several **kinds of truth** that rot at different speeds (the scalar-vs-array *decision* is stable; *PR #910 still OPEN* is not).
- Several **kinds of knowledge** that must not be mixed (how Lattice ApplyChange works vs “we used `--force-with-lease` on Thursday”).

A **folder** answers “what files belong to ENG-915519?”  
A **linear doc** answers “what happened in order?”  
A **chat** answers “what did we say this afternoon?”  
A **knowledge graph** answers “what is this connected to, and along which relation?”

That last question is the one a new agent (or a tired you in August) actually needs. “Products vs authoringScope” is not a subdirectory. It is an **edge**: *different field, different mutability, same Kronos epic.* If that edge is only in your head, every new chat re-derives it or gets it wrong (the Jira paste that still said authoringScope was an array).

### Why graphs beat the alternatives we actually tried

**Chat transcripts.** High fidelity, zero query. You cannot ask a transcript “show everything that touches Lattice identity rewrite” without reading it. Parallel chats do not merge. The agent in the mutation-guard session does not automatically know the shard-copy enrich/resolve model unless you paste or it greps.

**One giant markdown file.** We have that too: `iam-themis/.notes/PRODUCTS_FIELD_CONTEXT.md`. It is the right shape for **one initiative’s evidence** (SHAs, gates, D-numbers). It is the wrong shape for **cross-initiative navigation**. You do not want authoringScope, products, shard copy, and ETag retries as sections 47–90 of one file. Nobody reads section 47. They follow a link from the thing they already opened.

**Wiki-without-links (Confluence pages).** Pages exist; relationships are “related links” dumped in a footer, or search. Search is keyword. Keyword hits “NC” on every note. A wikilink `[[authoringScope]]` — `who authored` vs `[[products]]` — `what it applies to` is a **typed-enough** relation for humans and agents: you see the distinction in the link text on the index.

**Pure code / grep.** Necessary before you edit (VERIFY BEFORE YOU WRITE). Insufficient as memory: grep does not tell you that AccessPolicies were **deliberately removed** from products (D24) or that mutation checks must sit **above** persistence so ApplyChange still works. Those are decisions, not strings in `authoring_scope.go`.

**RAG over the whole vault.** That is a bag of chunks. Chunks do not preserve “this is decided / this is open / this is a process lesson / this is a SHA.” The graph’s job is to **force you to pick a node type** so retrieval is not “semantically nearby paragraphs.”

So: the graph is not decoration. It is how we store **relations and node types** that Jira, git, and chat each refuse to store well.

### What a node is, in this vault

Not “a file.” A file is the storage. The **node** is a thing you might want to arrive at from more than one direction:

| Node kind | Examples | Why it is a node |
| --- | --- | --- |
| Ticket | ENG-915519, ENG-949861 | Jira identity; has status, PRs, a worktree |
| Concept | authoringScope, lattice, global ACP | Shared by many tickets; definition must stay consistent |
| Product | NC, PC | Easy to collapse by acronym; must stay distinct |
| Repo | iam-themis | Ownership and “is this Themis-only?” |
| Person | Manish, Aditya, Praveen | Reviewer / design-lead routing |
| Pattern | audit-before-planning, ASCII tavern | How we fail; reused across tickets |
| Source | a specific index or overview note | Provenance; “where did this claim come from?” |

**Edges** are wikilinks, usually with display text that states the relation: “predecessor,” “sibling,” “mirrors the handler-layer validator,” “who-authored vs what-it-applies-to.”

If two tickets are related and you only put them in the same folder, you have encoded **containment**. Global IAM is not a containment hierarchy. Shard copy does not *contain* authoringScope. They **share** lattice and global ACP. That is a graph.

---

## 2. Why Obsidian (as a graph tool), decision by decision

Obsidian is not used here because it is trendy. It is used because it is a **local, file-native, bidirectional-link graph** over markdown we already write.

**Local files, not a web wiki.** The vault sits next to `nutanix-core`. Agents and `rg` can read it. No “export from Confluence.” MCP can read/write the same bytes you see in Graph view. If the company wiki is permissioned and HTML, the agent is blind.

**Markdown + wikilinks.** `[[../ENG-915519 - Global IAM authoringScope/00 - Index|ENG-915519]]` is both human-readable and machine-followable. Bidirectional backlinks mean you do not have to update every sibling when you add ENG-949861; the predecessor page already points forward, and Graph shows the reverse.

**Graph view is a debugging tool for the knowledge, not a toy.** If mutation guard does not show an edge to authoringScope, the *map* is wrong — the same way a missing import is wrong. We treat missing wikilinks as bugs in the notebook.

**One note ≠ one idea, on purpose.** A 12-file ticket is not vanity. It is so you can open **PR status** without loading **process lessons**, and so an agent can be told “read the index, then the spec, not the review rollup.” That is **progressive disclosure** for context windows. A graph with fat nodes (one 4,000-line note per ticket) recreates the giant `.notes` file we already have in git.

**Tags are secondary.** Frontmatter `tags: [eng-915519, kronos, moc]` is for filter and “this index is a map of content.” Tags do not replace links. `kronos` on five tickets does not tell you mutation guard **depends on** the authoringScope field existing on a branch.

**Why not only git `.notes/`?** Because `.notes/` is **canonical evidence per repo**. The vault is **cross-repo, cross-ticket navigation**. Duplicating SHAs into Obsidian is how notes go stale; the index **points** at `AUTHORING_SCOPE_CONTEXT.md` instead of copying it. Two stores, two jobs. That split is load-bearing.

**Why `inbox/` vs mixing everything at vault root?** Folders are a **weak** graph (tree). We still use a tree for *lifecycle*: raw ticket narrative stays together so “ingest this ticket” or “this is human-edited source” is a directory. The **meaning** graph is the links, not the folder nest. Putting all ENG-* folders in `inbox/` says “these are sources,” not “915519 is a child of Kronos.” Kronos is a concept node, not a parent folder.

---

## 3. Why Cursor is wired the way it is (reasoning, not steps)

**MCP to the vault.** The agent must follow wikilinks and edit indexes without you pasting files. If Cursor cannot see the vault, the graph exists only for you in Obsidian, and every new chat is pre-May. MCP is how the graph becomes **agent-addressable**.

**Absolute path in `mcp.json`.** The tool has no notion of “whatever vault I have focused in Obsidian.” Wrong path = parent vault without the plugin, which we already hit.

**Do not dump the vault into the prompt.** A graph is useful **because you traverse a few edges**, not because you embed every node. Traversing ENG-949861 → 915519 → scalar decision is ~3 notes. Embedding 40 inbox files is a bag again. The instruction “search related, then stop” is how you *use* a graph at inference time.

**Skills vs vault.** The graph stores **what is true and how nodes relate**. Skills store **what to do this turn** (audit, worktree, don’t invent SHAs). Mixing them — putting the verification ladder only as a wiki essay — means the agent might never open it. Mixing the other way — putting NC vs PC semantics only in a skill — means the skill rot when Kronos evolves. Concepts live in the graph; procedures live in skills; evidence lives in `.notes/`.

**alwaysApply rule.** A graph the agent is not required to start from is a hobby wiki. The rule is the **entry edge**: “you may not plan until you have walked related nodes.” That is not Obsidian’s feature. That is Cursor’s.

**alwaysApply ticket rules stay separate** (`iam-products-field.mdc`). Those are **constraints on a region of the graph** (products does not apply to APs). They are not a substitute for the graph. If you fold “APs out of scope” only into a wiki page, a products session might never open it. If you fold the whole Kronos map into that one rule, every chat pays for products law on a shard-copy ticket.

**Worktrees + `HEAD` check.** Parallel agents are concurrent writers on **code**. The vault is a concurrent writer on **maps**. Same class of bug: last writer wins unless you isolate. Worktrees isolate git; we do not yet isolate the vault per agent — so the discipline is “append decisions, don’t rewrite history; correction notes stay.”

---

## 4. How every issue is structured — and why each file exists

There is a **family resemblance**, not one rigid 12-file template. Small tickets are allowed to be small. Large ones split until a file has **one job**.

### 4.1 The index (`00 - Index.md`) is a map of content (MOC)

Every ticket that matters has an index. It is tagged `moc` when it is a map. Jobs:

1. **One-liner / blockquote** — what the ticket *is*, in one breath. If you cannot write this, you cannot link to it from other tickets.
2. **Status at a glance** — inverted pyramid. PRs, design lock, hard blocker, action needed. A returning human or agent must not wade through history to learn “still waiting on Manish’s swagger pin.”
3. **Read in this order** — numbered wikilinks. This is an **explicit path** through the subgraph. Graphs do not have a natural order; features do. The index supplies the order without flattening the graph into one document.
4. **Pointer to `.notes/`** — which repo file is canonical for SHAs/CI.
5. **PR URLs** — outbound to GitHub; not duplicated CI logs.
6. **Related work** — **typed** edges to other tickets (sibling, predecessor, “mirrors validator pattern”). This is the cross-ticket graph.

915519’s related-work line is the whole point of the graph in one sentence: *authoringScope is who-authored / immutable; products is what-it-applies-to / server-derived.* That distinction is an edge label. It does not belong only in Jira description.

949861’s index is **short** because the ticket is a follow-up. It does not copy the 12-file spec. It **depends on** 915519 (predecessor edge) and **mirrors** 910350’s handler-layer pattern (analogy edge). Structure scales down: one implementation note, not twelve.

910350’s index states **repo scope: Themis only** on the map. That is an edge to “do not open ntnx-api-iam.” A folder named `ENG-910350` cannot say that; a sentence on the index can.

### 4.2 Split files by *kind of truth*

Look at 915519 / 932537 — the large template:

| File | Kind of truth | Rot rate | Who reads it |
| --- | --- | --- | --- |
| `01 Overview` | Meaning and why | Slow | Everyone first |
| `02 Canonical spec` | Contract (shape, mutability) | Slow after lock | Implementers, reviewers |
| `03 Repository map` | Ownership | Medium | “which repo do I touch?” |
| `04 PR status` | Live GitHub | **Fast** | Must re-verify; treated as cache |
| `05 Cross-repo flow` | How a bit moves | Slow | Debugging empty API fields |
| `06 Storage & migration` | DB, indexes, backfill | Medium | Storage work |
| `07 Handler logic` | Runtime stamping/guards | Medium | Themis |
| `07 API surface` (products) | Which APIs expose it | Medium | Review / tests |
| `08 PR reviews rollup` | Comment inventory | Fast | “certain enough to act?” |
| `09 Open questions` | **Not decided** | Fast | Must not mix with 10 |
| `10 Decisions log` | **Decided**, with why | Append-only | Future agents |
| `11 Process lessons` | Abstract how-we-work | Slow | Other tickets |
| `12 Issues encountered` | Concrete incident receipts | Append-only | “we already burned this” |
| `12 PR drafts` (products) | Paste for GitHub | Fast | Bodies otherwise die on chat reset |

**The load-bearing split is 09 vs 10** (open vs decided). Mixing them is how every session re-argues scalar vs array. The graph encodes that as **two nodes**, not a highlight color in one essay.

**11 vs 12** is abstract vs concrete. Process lessons are **reusable graph nodes** (verification ladder). Incidents are **evidence hanging off a ticket**. If you only keep incidents, the next ticket does not find “audit before plan.” If you only keep abstracts, you cannot defend the lesson. Both exist; 11 should eventually be **promoted** to `wiki/patterns/` so they are not trapped inside ENG-932537’s folder. That promotion is how a ticket-local graph becomes a vault-global graph.

### 4.3 Smaller tickets use a subset

910347 / 910350:

- Overview & design / requirements  
- Implementation & testing / verification  
- Decisions log  
- Open questions & status  

Same **kinds**, fewer slices. Overview still has why (lattice + local identity). Decisions still append-only. Open still separate.

949861: **one** implementation note because the requirement is one guard, and the interesting content is decisions + the AllowOperation finding. Forcing 12 files would be fake structure.

**Rule:** add a file when you have a **new kind of truth** or a **file you must open without the others**. Do not add files to look complete.

### 4.4 Numbered prefixes

`00`, `01`, … are **sort order in the file explorer**, not graph order. Graph order is the index’s numbered list. Prefixes keep Finder/Obsidian file list aligned with “read this first” for humans who never open Graph view. Agents should follow **wikilinks**, not directory sort — but humans live in the file tree, so the tree is sorted to match the path.

### 4.5 Frontmatter

- `tags` including ticket id: query “all kronos” or “all moc.”  
- `status`: in-progress / implemented-not-committed — a **property on the ticket node**, visible without reading the body.  
- `updated`: honesty about staleness. Combined with “re-verify GitHub,” it tells you the PR status section is a cache.

### 4.6 Cross-ticket edges (the actual Kronos graph)

From the technical-stack map and indexes:

```
ENG-915519 authoringScope
    → predecessor of ENG-949861 mutation guard
    → sibling of ENG-932537 products (contrast edge: who vs what)
ENG-910344 RPC resolvers
    → parent of ENG-910347 shard copy (enrich/resolve extended to bulk)
ENG-910347
    → cited by ENG-910350 (local identity has no stable federated id)
ENG-910350
    → validator pattern cited by ENG-949861 (handler-layer, not persistence)
```

Plus **concept** hubs: lattice, global ACP, NC, PC. Tickets hang off concepts so a search for “lattice” does not require remembering ticket numbers.

This is why NC and PC are **separate nodes**. Collapsing them into “Nutanix” destroys the mutation-guard predicate (PC request + NC-authored).

### 4.7 The technical-stack note is a hub, not a ticket

`Nutanix Technical stack.md` is a **high-fan-in node**: platform sketch + issue relationship map + troubleshooting catalog + “before the next issue.” It is allowed to be long because it is a **hub**. Tickets should **link out** to a troubleshooting pattern rather than each copying the `$fv` sermon. Hubs that never link to tickets become blogs. This hub’s job is edges into ENG-*.

---

## 5. How you are supposed to *traverse* the graph (agent and human)

This is the usage logic that makes the structure worth the cost.

**Start at the index of the ticket you were asked about**, not at the hub, unless you do not know the ticket id.

**Walk related-work links** until you can say which lessons apply and which **feature-specific decisions do not** (products ≠ APs is not a shard-copy law).

**Open `.notes/` in the repo the index names** for evidence. Do not treat vault PR status as live.

**Grep code** before planning (audit). The graph told you *where* and *what was decided*; grep tells you *what is in HEAD*.

**Write back on the correct node type:**

- New SHA / test command → `.notes/`  
- Status / blocker → index (fast-rot section)  
- New decision → decisions log (append)  
- New incident → issues encountered  
- Reusable how-to → process lessons, then promote to `wiki/patterns/`  
- Stable definition → concept page if you maintain that layer  

**Never** append a SHA to a concept page. **Never** put “still OPEN” on a canonical spec. Wrong node type is how the graph lies.

---

## 6. Presenting this (graph-first talk)

Skip plugin features. Show:

1. Jira list of five tickets (flat).  
2. The same five in Graph view (edges).  
3. Click 949861 → 915519 → “scalar, not array.”  
4. Open `09` vs `10` on 915519: open vs decided.  
5. Open 910350 index: “Themis only.”  
6. Open technical-stack hub, jump to a troubleshooting pattern, then to the ticket that earned it.  
7. New Cursor chat: “related nodes only, then grep.”  

The audience should leave believing **links are the data structure**, folders are packaging, and Cursor is a walker of that graph plus a writer that must put ink on the right node.
