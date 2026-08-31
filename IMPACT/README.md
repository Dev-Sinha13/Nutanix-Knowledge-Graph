---
title: Company impact — Nutanix deliverables
type: moc
created: 2026-08-31
updated: 2026-08-31
audience: human (GitHub + Obsidian)
---

# Company impact — Nutanix IAM / Kronos deliverables

**This folder is the human-readable impact record.** It is separate from the engineering wiki (`inbox/`, `wiki/`) on purpose.

Author: **Dev Sinha**  
Program: **Kronos / Global IAM** (Nutanix Central federation)  
Window covered: **May–August 2026**

> **Staleness.** Ticket status, PR checks, and SHAs are from vault notes on the dates in each file. Re-verify GitHub before treating merge/CI as current.

---

## How each ticket page is structured

Every deliverable page uses the same sections:

1. **The problem** — what was broken or missing for the product  
2. **How it was solved** — the design that landed  
3. **Impact** — what the company / NC / customers got  
4. **How it was done** — the actual engineering process (repos, order, verification)  
5. **Technologies and skills** — languages, systems, and practices used  
6. **Significant technical difficulties** — only load-bearing ones (design traps, federation, schema/CI coupling). Not one-off debugger noise.

---

## Why this work exists

Nutanix Central (NC) is a **global management plane** over Prism Central (PC) clusters. Global IAM entities — Roles and Access Control Policies — are **lattice-replicated** so a customer can author once on NC and have authorization mean the same thing on every joined PC. That only works if the system knows **who authored** an object, **which products it applies to**, **which humans it names** after copy, and **who is allowed to change it**.

---

## Deliverables

| Ticket | Problem in one line | Page |
| --- | --- | --- |
| ENG-915519 | Global Roles/ACPs had no durable origin | [ENG-915519](ENG-915519.md) |
| ENG-932537 | NC could not filter federated IAM by product | [ENG-932537](ENG-932537.md) |
| ENG-949861 | PC could still mutate NC-authored globals (or get a misleading error) | [ENG-949861](ENG-949861.md) |
| ENG-910344 | ACP user/group UUIDs are cluster-local; copy is wrong | [ENG-910344](ENG-910344.md) |
| ENG-910347 | New PC join (shard copy) still stored unresolved UUIDs | [ENG-910347](ENG-910347.md) |
| ENG-910350 | Global ACPs could grant to local users / service accounts | [ENG-910350](ENG-910350.md) |
| ENG-948242 | NC could not search entities by attribute metadata | [ENG-948242](ENG-948242.md) |
| ENG-953364 | Brownfield products migration could mark complete with zero tenants | [ENG-953364](ENG-953364.md) |
| ENG-956163 | ETag races during that migration could skip roles or lie | [ENG-956163](ENG-956163.md) |
| ENG-924709 | Global-role delete guard must not be confused with durable completion | [ENG-924709](ENG-924709.md) |
| Practice | How the program compounded | [engineering-practice](engineering-practice.md) |

Engineering sources: `inbox/ENG-*/` for tickets with full notes; [Nutanix Technical stack](../inbox/Nutanix%20Technical%20stack.md) for 948242 / 953364 / 956163 / 924709.

---

## How the tickets form one product

```
ENG-910344  enrich/resolve on ApplyChange          [merged iam-themis #1535]
    └── ENG-910347  same rewrite on shard-copy bootstrap
ENG-910350  do not author global ACPs on local identities

ENG-915519  authoringScope field (who authored)
    └── ENG-949861  mutation guard (PC cannot change NC-authored)

ENG-932537  products field (what it applies to)
    └── ENG-953364  brownfield migration completion
    └── ENG-956163  ETag / retry during that migration

ENG-924709  global-role deletion guardrail (do not confuse with 953364)
ENG-948242  entity search metadata (NC findability)
```

**Who vs what.** `authoringScope` is provenance. `products` is membership. Do not key edit-permission off `products`.  
**API vs lattice.** Guards on the handler so Lattice can still sync.  
**NC authors, PC consumes.** Guards are directional.
