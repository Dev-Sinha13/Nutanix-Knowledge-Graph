---
tags: [eng-915519, kronos, overview]
status: stable
---

# Overview — what authoringScope is and why

## What

`authoringScope` is a **read-only** field on global IAM Access Control Policies (ACPs) and Roles. It records the **product cluster on which the entity was originally created** — currently one of:

- `NC` — Nutanix Cloud Manager
- `PC` — Prism Central

## Why

Part of the broader **Kronos** / Global IAM initiative to:

- Distinguish which cluster authored a global entity in a federated / multi-cluster environment
- Enable downstream UIs to display origin and gate edit permissions accordingly
- Support future federation patterns where an entity might be co-authored by multiple products (hence array, not scalar)

A "global" entity is one that is lattice-replicated across clusters. A non-global tenant-specific entity does NOT need authoringScope — only globals do.

## What it's NOT

- It is NOT the same as `products` (ENG-932537). `products` lists which products an entity *applies to*; `authoringScope` lists *who authored it*. Independent concerns, different fields, different semantics. See [[../ENG-932537 - IAM products field/00 - Index|ENG-932537 vault]] for the sibling implementation.
- It is NOT user-supplied. Clients can read it but never write it. The server stamps it at create time.
- It is NOT mutable. Once set, it stays set for the lifetime of the entity. UPDATE statements deliberately skip this column.

## Who's involved

- **Manish Lokur** — Design lead for the Kronos initiative; owns the array-vs-string question and the go-swagger version pin
- **Aditya** — Primary reviewer on ntnx-api-iam and iam-utils; granted write access to iam-utils on 2026-05-26
- **Praveen** — Primary reviewer on iam-themis
- **Dev (you)** — Implementing across all three repos

## See also

- [[02 - Canonical Spec]] for the formal field shape
- [[07 - Handler Logic]] for how the server actually stamps the value
- Sibling Kronos initiative: [[../ENG-932537 - IAM products field/00 - Index|ENG-932537 — IAM `products` field]] (different field, different semantics, much overlap in repo layout and review process patterns)
- Up: [[00 - Index]]
