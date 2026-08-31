---
title: Cursor User Rules
type: cursor-rule
created: 2026-08-31
updated: 2026-08-31
live_path: Cursor Settings → General → Rules for AI (User Rules)
---

# User rules (all projects)

These are the user-level instructions Cursor injects into every chat. They are **not** files under `~/.cursor/rules/` as of 2026-08-31. Edit in Cursor Settings, or paste into User Rules after changing this note.

Index: [[cursor/00 - Index]]

## Committing changes with git

Only create commits when requested. If unclear, ask first.

- NEVER update the git config
- NEVER run destructive/irreversible git commands (like push --force, hard reset, etc) unless explicitly requested
- NEVER skip hooks (`--no-verify`, `--no-gpg-sign`, etc) unless explicitly requested
- NEVER force push to main/master; warn if requested
- Avoid `git commit --amend` unless: user asked, or a hook auto-modified files after a commit **you** just created that has **not** been pushed
- If commit FAILED or was REJECTED by hook, NEVER amend — fix and create a NEW commit
- If already pushed, NEVER amend unless the user explicitly asks (requires force push)
- NEVER commit unless the user explicitly asks
- Do not commit secrets (`.env`, `credentials.json`, …)
- Pass commit messages via HEREDOC
- Never use git commands with the `-i` flag (`git rebase -i`, `git add -i`)
- Do not push unless the user asks
- Parallel git status / diff / log before committing; follow this repo’s commit message style
- Commit message: 1–2 sentences on **why**, not what

## Creating pull requests

Use `gh` via the shell for all GitHub tasks (issues, PRs, checks, releases). If given a GitHub URL, use `gh` to fetch what you need.

Before creating a PR, in parallel: `git status`, `git diff`, remote tracking / up to date, `git log` and `git diff [base]...HEAD` for the full branch history.

Then: create branch if needed, push with `-u` if needed, `gh pr create` with HEREDOC body:

```
## Summary
<1-3 bullet points>

## Test plan
[Checklist]
```

Return the PR URL. NEVER update git config. Do not use TodoWrite or Task tools for this flow.

## Web UI verification

When implementing or fixing anything in a web application (UI, layout, styling, routing, client state, or rendered data), verify in the browser before declaring complete.

- Exercise the changed feature end to end the way a real user would
- A single screenshot is not verification
- Check every page/route that shares the state, data, or components you touched
- Hunt for regressions on surrounding flows
- Verify empty/error/flag variants, not only the main path
- For layout changes, consider desktop and mobile viewports
- If verification finds a problem, fix and re-verify
- If no browser tools: tests, curl, or render scripts — and say what you could not verify

## Related

- Git commits: subject-only; bodies live in `.notes/` / PR. Strip Cursor `Co-authored-by` with `git commit-tree`, never `--no-verify`. See [[cursor/skills/iam-ticket-workflow]].
- Gerrit: [[cursor/rules/gerrit-push-for-review]]
