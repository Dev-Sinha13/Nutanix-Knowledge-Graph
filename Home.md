---
title: Nutanix-Core home
type: moc
created: 2026-08-30
---

# Nutanix-Core

Vault layout is tuned for **Karpathy LLM Wiki**: raw notes stay in `inbox/`, generated knowledge lives in `wiki/`.

## Company impact (GitHub + humans)

**Read this first if you want what shipped and why it matters to Nutanix:** [[IMPACT/README|IMPACT/]]. Separate folder from the engineering wiki on purpose.


## Start here

Follow [[How to set this up]] for the live setup demo (Obsidian → plugin → folders → Cursor MCP → WikiSkill). **Editable Cursor rules, skills, and MCP:** [[cursor/00 - Index]]. Operational setup for this machine: [[cursor/setup]]. Speaker notes for a talk: [[Presentation outline]]. Long-form talk + 4-month instruction clone: [[Deep presentation and full clone]]. Why the graph exists and how tickets are structured: [[Why a knowledge graph]].

1. Configure an LLM: **Settings → Karpathy LLM Wiki → LLM Provider** → Test Connection → Save.
2. Ingest sources: command palette → `Karpathy LLM Wiki: Ingest from folder` → choose `inbox`.
3. Ask questions: ribbon chat bubble, or `Karpathy LLM Wiki: Query Wiki`.

## Folders

| Folder | Role |
| --- | --- |
| [[inbox/README\|inbox/]] | Source notes. The plugin watches this folder. |
| [[wiki/Welcome to Karpathy LLM Wiki\|wiki/]] | Generated entity, concept, and source pages. Do not hand-edit unless you mean it. |
| [[wiki/schema/config]] | Controlled vocabulary injected into ingest prompts. |
| [[cursor/00 - Index\|cursor/]] | Working copies of Cursor rules, skills, MCP. Edit here, copy back to `~/.cursor/` or the repo. |

## Source tickets

- [[inbox/ENG-915519 - Global IAM authoringScope/00 - Index|ENG-915519 — authoringScope]]
- [[inbox/ENG-949861 - Authoring scope mutation guard/00 - Index|ENG-949861 — mutation guard]]
- [[inbox/ENG-910347 - Resolver Shard ACP/00 - Index|ENG-910347 — resolver shard ACP]]
- [[inbox/ENG-910350 - Global ACP external identities/00 - Index|ENG-910350 — global ACP identities]]
- [[inbox/ENG-932537 - IAM products field/00 - Index|ENG-932537 — products]]
- [[inbox/Nutanix Technical stack|Nutanix technical stack]]

## WikiSkill layers

| Layer  | Where                                                                      | Use                                                         |
| ------ | -------------------------------------------------------------------------- | ----------------------------------------------------------- |
| Raw    | `inbox/`, repo `.notes/`                                                   | Evidence. Do not overwrite.                                 |
| Wiki   | [[wiki/index]] including [[wiki/patterns/audit-before-planning\|patterns]] | Compounding knowledge. Never roll back.                     |
| Skills | `~/.cursor/skills/iam-ticket-workflow`, `wikiskill-loop` — vault copies in [[cursor/00 - Index\|cursor/]] | Procedures the agent runs. Gated via [[wiki/skill-impact]]. |

While solving a ticket, follow **skills**. At the end of a real turn, update **wiki patterns** + `.notes/`. Do not dump the whole wiki into the agent prompt.

Plugin **Query Wiki** still needs an LLM key. Asking Cursor is the substitute until then.
