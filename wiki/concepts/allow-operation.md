---
type: concept
created: 2026-08-30
tags: [method]
aliases: [AllowOperation]
sources:
  - "[[sources/eng-949861-implementation]]"
---

# AllowOperation

## Definition
Existing lattice util (`services/utils/lattice_utils.go`) that returns false on PC for `isGlobal && caller==CallerAPI`, causing six create/update/delete handlers to reject with HTTP 400 "Global ... are only allowed for NC products".

## Key Characteristics
- Already blocks NC-authored (global) API mutations on PC
- The ENG-949861 guard sits **before** this check to emit a 403 ownership error instead
- Lattice sync callers are not `CallerAPI`, so they still pass

## Applications
- Baseline Global IAM product limitation on PC
- Must not be the only long-term control if that blanket block is relaxed

## Related Concepts
- [[concepts/mutation-guard|mutation guard]]
- [[concepts/global-acp|global ACP]]

## Related Entities
- [[entities/eng-949861|ENG-949861]]
- [[entities/iam-themis|iam-themis]]

## Mentions in Source
- "`AllowOperation(isGlobal, caller)` returns **false** on PC for `isGlobal && caller==CallerAPI`." — [[sources/eng-949861-implementation]]
