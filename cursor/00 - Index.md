---
title: Cursor rules and setup
type: moc
created: 2026-08-31
updated: 2026-08-31
---

# Cursor rules and setup

Working copies of the Cursor config this vault depends on. Edit here, then copy the fenced file body back to `live_path`. Cursor does **not** read these vault notes at inference time.

Live machine paths (this laptop, 2026-08-31):

| This note | Live file | Scope |
| --- | --- | --- |
| [[cursor/setup]] | (checklist, not a single file) | How MCP + rules + skills are wired |
| [[cursor/mcp]] | `~/.cursor/mcp.json` | User MCP |
| [[cursor/rules/wikiskill-loop]] | `~/.cursor/rules/wikiskill-loop.mdc` | User, `alwaysApply` |
| [[cursor/rules/iam-products-field]] | `nutanix-core/.cursor/rules/iam-products-field.mdc` | Repo, `alwaysApply` |
| [[cursor/rules/iam-tavern-local-k3d]] | `nutanix-core/.cursor/rules/iam-tavern-local-k3d.mdc` | Repo, tavern YAML globs |
| [[cursor/rules/gerrit-push-for-review]] | workspace / user rule (Gerrit `refs/for`) | Always on in this workspace |
| [[cursor/rules/user-rules]] | Cursor Settings → User Rules | All projects |
| [[cursor/skills/wikiskill-loop]] | `~/.cursor/skills/wikiskill-loop/SKILL.md` | User skill |
| [[cursor/skills/iam-ticket-workflow]] | `~/.cursor/skills/iam-ticket-workflow/SKILL.md` | User skill |
| [[cursor/skills/iam-ticket-workflow-PURPOSE]] | `~/.cursor/skills/iam-ticket-workflow/PURPOSE.md` | Skill → patterns |

Demo click-path (talk script, not the live files): [[How to set this up]].

## Apply an edit

1. Change the fenced source in the matching note.
2. Copy that fence into the `live_path` (or ask Cursor: “sync this cursor/ note back to the live file”).
3. If you changed a skill, append [[wiki/skill-impact]].
4. Restart Cursor or reload windows after `mcp.json` changes.

## Gaps on this machine

- `wikiskill-loop.mdc` is **user-global only**. It is not yet in `nutanix-core/.cursor/rules/`. Cloud / teammate clones will miss the loop unless you copy it into the repo.
- `~/.cursor/mcp.json` currently lists the **parent** vault (`.../Obsidian Vault`) and `hackathon/vault`, not `Nutanix-Core`. Wrong path was the original MCP footgun. See [[cursor/mcp]].
