# Nutanix Knowledge Graph

Obsidian vault + human **company-impact** writeup for **Dev Sinha**’s Kronos / Global IAM work at Nutanix (May–August 2026).

This GitHub repo is the knowledge graph: ticket maps, concepts, Cursor setup copies, and a separate impact section you can read without opening Obsidian.

---

## Start here if you want impact on the company

**→ [`IMPACT/`](IMPACT/README.md)** (separate folder, easy to spot)

That section is written for humans. **Every ticket page uses the same outline:** problem, how it was solved, impact, how it was done, technologies and skills, significant technical difficulties (design/federation/schema — not one-off debugger noise).

| Ticket | One-line company outcome |
| --- | --- |
| [ENG-915519](IMPACT/ENG-915519.md) | Global Roles/ACPs remember **who authored** them (`NC` vs `PC`) |
| [ENG-932537](IMPACT/ENG-932537.md) | NC can **filter IAM by product** (`products` on Entities and Roles) |
| [ENG-949861](IMPACT/ENG-949861.md) | PC cannot clobber **NC-managed** Roles/ACPs (403) |
| [ENG-910344](IMPACT/ENG-910344.md) | Federated ACP identities: enrich on NC, resolve on PC (**merged** #1535) |
| [ENG-910347](IMPACT/ENG-910347.md) | Same rewrite on **cluster-join shard copy**, not only later edits |
| [ENG-910350](IMPACT/ENG-910350.md) | Global ACPs cannot grant to **local users / service accounts** |
| [ENG-948242](IMPACT/ENG-948242.md) | v4 Entity **search metadata** (live PC verified) |
| [ENG-953364](IMPACT/ENG-953364.md) | Brownfield products migration: **empty tenant list ≠ complete** |
| [ENG-956163](IMPACT/ENG-956163.md) | ETag races retry honestly; completion stays true |
| [ENG-924709](IMPACT/ENG-924709.md) | Global-role delete guardrail ≠ durable migration flag |
| [Practice](IMPACT/engineering-practice.md) | Notes, rules, and skills so the program compounds |

PR merge/CI state in those pages is **as recorded in the vault**. Re-check GitHub before treating it as current.

---

## Rest of the graph (engineering)

| Path | Role |
| --- | --- |
| `inbox/` | Source ticket notes (indexes, specs, PRs, decisions) |
| `wiki/` | Concepts, entities, patterns (Obsidian Graph) |
| `cursor/` | Copies of Cursor rules, skills, MCP setup |
| [How to set this up.md](How%20to%20set%20this%20up.md) | Live demo: Obsidian then Cursor |
| [Home.md](Home.md) | Vault dashboard (Obsidian) |
| [COMPLETE-GUIDE.txt](COMPLETE-GUIDE.txt) | Linear full dump |

Open the folder as an **Obsidian vault** (this directory only — not a parent vault). Do not nest vaults.

Private repo. Do not commit API keys, Artifactory passwords, or raw CI logs with credentials.
