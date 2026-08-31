---
tags: [eng-949861, kronos, global-iam, authoring-scope, index]
status: implemented-not-committed
updated: 2026-07-09
---

# ENG-949861 — Do not allow changes to entities depending on authoring scope

> Reject user-initiated **update/delete** of an NC-authored Role or Authorization Policy when the request runs on a **PC** cluster. Follow-up enforcement for [[../ENG-915519 - Global IAM authoringScope/00 - Index|ENG-915519]], which added the read-only `authoringScope` field but no mutation gating.

## Contents
1. [[01 - Implementation & Decisions]] — requirement, the AllowOperation finding, decisions, files, verification, reproduce.

## Status at a glance
- Branch `ENG-949861-authoring-scope-mutation-guard`, worktree `.wt-themis-949861`, based off `ENG-915519-authoring-scope-handler` (`a41faa5cf`).
- Implemented + all local gates green. **Not committed, not pushed.**
- Depends on unmerged ENG-915519 (the `authoringScope` field only exists on that branch).

## See also
- [[../ENG-915519 - Global IAM authoringScope/00 - Index|ENG-915519 — authoringScope field]] (predecessor)
- [[../ENG-910350 - Global ACP external identities/02 - Implementation & Verification|ENG-910350]] — the handler-layer validator pattern this mirrors
