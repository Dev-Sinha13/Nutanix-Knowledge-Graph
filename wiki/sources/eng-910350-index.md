---
type: source
tags: [notes]
created: 2026-08-30
updated: 2026-08-30
source_note: "[[inbox/ENG-910350 - Global ACP external identities/00 - Index]]"
sources:
  - "[[entities/eng-910350]]"
  - "[[concepts/identity-federation]]"
---

# ENG-910350 index (source)

## Summary
Authoring-side guardrail: when creating or updating a global ACP via v4, reject it if any concretely referenced user or group is locally managed (LOCAL user or SERVICE_ACCOUNT). Only externally managed identities may appear on a lattice-replicated ACP.

## Key Points
- Local identities have no stable cross-cluster identity
- iam-themis only; reuses vendored IAMAuthn SDK client
- Sibling to resolver shard work (ENG-910347)

## Mentioned Pages
- [[entities/eng-910350|ENG-910350]]
- [[entities/eng-910347|ENG-910347]]
- [[entities/iam-themis|iam-themis]]
- [[concepts/identity-federation|externally-managed identities]]
- [[concepts/global-acp|global ACP]]
- [[concepts/lattice|lattice]]
