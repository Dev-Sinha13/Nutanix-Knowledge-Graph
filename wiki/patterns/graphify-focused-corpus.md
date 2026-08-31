---
title: Graphify focused corpus
type: pattern
created: 2026-08-30
tags: [strategy]
---

# Graphify focused corpus

## Failure mode
Running Graphify on four IAM repo roots (including `vendor/`) produces thousands of files. Intra-repo AST edges drown the inter-repo `go.mod` `depends_on` chain. `graphify merge-graphs` prefixes node ids per repo, which **splits** canonical package ids that `manifest_ingest` otherwise merges.

## Strategy
Build one corpus with all four `go.mod` files plus the integration surface (handlers, storage, configload, v4 YAML, Themis docs). Exclude `vendor/`, tests, and generated protobufs. Extract once so package nodes stay `pkg:<module>` and `depends_on` edges connect Themis → utils → ntnx-api-iam.

## Workaround
Vault: `/Users/dev.sinha/Documents/Obsidian Vault/IAM-Repos-Graph` (not Nutanix-Core). Rebuild from `_corpus/`. Open that folder as an Obsidian vault; start at `00-Start-here.md`.

## Evidence
- 2026-08-30: 106 files → 2161 nodes, 4640 edges. `graphify path` themis → ntnx-api-iam is 1-hop EXTRACTED `depends_on`.
