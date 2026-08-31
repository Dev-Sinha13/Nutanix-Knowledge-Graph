---
title: Company impact — Nutanix deliverables
type: moc
created: 2026-08-31
audience: human (GitHub + Obsidian)
---

# Company impact — Nutanix IAM / Kronos deliverables

**This folder is the human-readable impact record.** It is separate from the engineering wiki (`inbox/`, `wiki/`) on purpose: those pages are how an agent walks related work; these pages are how a person sees **what shipped, why the company needed it, and what would be broken without it.**

Author: **Dev Sinha**  
Program: **Kronos / Global IAM** (Nutanix Central federation)  
Window covered: **May–August 2026**  
Repos: `ntnx-api-iam`, `iam-utils`, `iam-themis`, `iam-bootstrap`, plus coordination with `iam-user-authn` and `iam-deployment`

> **Staleness.** Ticket status, PR checks, and SHAs below are copied from vault notes last updated on the dates in each file. Re-verify GitHub before treating merge/CI as current. Do not invent newer SHAs.

---

## Why this work exists (one paragraph)

Nutanix Central (NC) is a **global management plane** over Prism Central (PC) clusters. Global IAM entities — Roles and Access Control Policies — are **lattice-replicated** so a customer can author once on NC and have authorization mean the same thing on every joined PC. That only works if the system knows **who authored** an object, **which products it applies to**, **which humans it names** after copy, and **who is allowed to change it**. Every deliverable in this folder closes one of those holes. Together they are the difference between a federated IAM product and a set of per-cluster IAM islands that look global in the UI and are wrong in the data.

---

## Impact at a glance

| Ticket | What the company got | Without it | Depth |
| --- | --- | --- | --- |
| [ENG-915519](ENG-915519.md) | Provenance: global Roles/ACPs stamped `NC` or `PC`, immutable, readable | NC UI cannot show origin; mutation guard has nothing to key on | [inbox](../inbox/ENG-915519%20-%20Global%20IAM%20authoringScope/00%20-%20Index.md) |
| [ENG-932537](ENG-932537.md) | Topology: Entities and Roles carry `products` so NC can filter federated IAM by product | NC lists every IAM object globally with no product tag | [inbox](../inbox/ENG-932537%20-%20IAM%20products%20field/00%20-%20Index.md) |
| [ENG-949861](ENG-949861.md) | Ownership: PC cannot PUT/DELETE an NC-authored Role/ACP (403, clear message) | PC operators can appear to “edit global IAM” or get a misleading 400 | [inbox](../inbox/ENG-949861%20-%20Authoring%20scope%20mutation%20guard/00%20-%20Index.md) |
| [ENG-910344](ENG-910344.md) | Identity federation model: enrich on NC, resolve on PC (incremental ApplyChange, **merged**) | Global ACP user/group UUIDs are cluster-local and wrong after copy | parent of 910347 |
| [ENG-910347](ENG-910347.md) | Same rewrite on **bulk shard-copy bootstrap** when a PC first joins | A newly joined PC stores unresolved UUIDs until a later ApplyChange | [inbox](../inbox/ENG-910347%20-%20Resolver%20Shard%20ACP/00%20-%20Index.md) |
| [ENG-910350](ENG-910350.md) | Authoring guard: global ACPs may only name **externally managed** identities | Global grants to local users/SAs replicate as meaningless / dangerous | [inbox](../inbox/ENG-910350%20-%20Global%20ACP%20external%20identities/00%20-%20Index.md) |
| [ENG-948242](ENG-948242.md) | Backend entity-search metadata on v4 Entity GET/LIST (six fields, live PC verified) | NC/search cannot query IAM entities by the attributes the UI needs | technical stack |
| [ENG-953364](ENG-953364.md) | Brownfield `products` migration **completion contract** (empty tenant list ≠ done) | Migration can mark complete while tenants were never processed | technical stack |
| [ENG-956163](ENG-956163.md) | Optimistic concurrency: ETag mismatch is a real conflict; Bootstrap retries the whole job | Stale ETag retries skip roles or loop locally and lie about completion | technical stack |
| [ENG-924709](ENG-924709.md) | Configurable global-role deletion guardrail (in-memory runtime flag) | Confusing this flag with durable migration completion would ship a false “done” | technical stack |

Process / compounding impact (how later tickets got cheaper): [engineering-practice.md](engineering-practice.md).

---

## How the tickets form one product, not ten chores

```
ENG-910344  enrich/resolve on ApplyChange          [merged iam-themis #1535]
    └── ENG-910347  same rewrite on shard-copy bootstrap
ENG-910350  do not author global ACPs on local identities
            (cites 910347: local UUID has no federated meaning)

ENG-915519  authoringScope field (who authored)
    └── ENG-949861  mutation guard (PC cannot change NC-authored)

ENG-932537  products field (what it applies to)
    └── ENG-953364  brownfield migration completion
    └── ENG-956163  ETag / retry during that migration

ENG-924709  global-role deletion guardrail (do not confuse with 953364)

ENG-948242  entity search metadata (NC findability)
```

**Who vs what.** `authoringScope` is provenance (one cluster, immutable). `products` is membership (multi-value, server-derived on roles). Keying edit-permission off `products` would be a product bug: a role tagged `["NC","PC"]` can still be locally editable on PC if it is not global.

**API vs lattice.** Mutation guards run on the **API handler** so Lattice `ApplyChange` can still update PC copies. If you persist the block in storage, federation dies.

**NC authors, PC consumes.** That direction is load-bearing. Guards are NC→PC, not symmetric.

---

## Company-facing outcomes (plain language)

1. **Nutanix Central can show and filter IAM in a multi-product world.** Without `products` and search metadata, the global UI is a flat dump of every cluster’s IAM objects.
2. **Global roles and ACPs have an origin.** Operators and UIs can tell “this was authored on NC” from “this was authored on this PC.”
3. **PC local admins cannot clobber NC policy.** The 403 is an ownership rule, not a vague product limitation.
4. **Permissions survive cluster join.** Shard copy no longer copies the wrong user/group UUIDs; incremental sync already did this after #1535.
5. **You cannot publish a “global” grant to a local user or service account.** Those identities do not exist on other clusters.
6. **Brownfield clusters can be migrated to `products` without a false completion.** Empty tenant lists and ETag races were real ways to mark done while data was wrong.
7. **The next engineer (or agent) does not start from zero.** Ticket indexes, decision logs, and Cursor skills are part of the deliverable — they are how a four-month multi-repo program stays coherent.

---

## Repos and the public-field chain

For any **public IAM API field**, the ownership chain is:

1. `ntnx-api-iam` — public v4 schema and generated models  
2. `iam-utils` — internal Swagger / shared helpers  
3. `iam-themis` — AuthZ handlers, storage, Lattice, most Tavern tests  
4. `iam-bootstrap` — startup seeding and orchestration  
5. `iam-user-authn` — tenants and identity enrich/resolve  
6. `iam-deployment` — CI / k3d / deployed images  

Not every ticket needs all six. ENG-910347 and ENG-910350 are Themis-only. ENG-932537 and ENG-948242 needed the full schema → service → bootstrap (and deployment) coordination.

---

## How to read this repo

| If you want… | Open |
| --- | --- |
| Company impact (this folder) | `IMPACT/` — start here on GitHub |
| Ticket maps, PRs, decisions | `inbox/ENG-*/` |
| Concepts and Graph | `wiki/` |
| Cursor rules/skills copies | `cursor/` |
| Live setup demo | [How to set this up.md](../How%20to%20set%20this%20up.md) |
| Full linear dump | [COMPLETE-GUIDE.txt](../COMPLETE-GUIDE.txt) |

Obsidian: [[Home]] · GitHub: [README.md](../README.md)
