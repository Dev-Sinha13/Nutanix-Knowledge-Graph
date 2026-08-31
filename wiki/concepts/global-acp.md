---
type: concept
created: 2026-08-30
tags: [term]
aliases: [ACP, Authorization Policy, Access Control Policy]
sources:
  - "[[sources/eng-910350-index]]"
  - "[[sources/eng-915519-overview]]"
---

# global ACP

## Definition
An Access Control Policy created with `isGlobal=true` and lattice-replicated across clusters. Global ACPs carry `authoringScope` and may not reference cluster-local identities.

## Key Characteristics
- Authoring is a v4 concern (v1/proxy persist locally and do not propose to the lattice)
- Must use externally managed identities (SAML / LDAP / IDP, NC-propagated groups)
- NC-authored globals are read-only on PC at the API layer

## Applications
- Federated authorization across NC and PC
- Guardrails: [[concepts/identity-federation|external-identity restriction]] and [[concepts/mutation-guard|mutation guard]]

## Related Concepts
- [[concepts/authoring-scope|authoringScope]]
- [[concepts/identity-federation|externally-managed identities]]
- [[concepts/lattice|lattice]]

## Related Entities
- [[entities/eng-910350|ENG-910350]]
- [[entities/eng-910347|ENG-910347]]
- [[entities/eng-915519|ENG-915519]]
- [[entities/eng-949861|ENG-949861]]

## Mentions in Source
- "When an Access Control Policy is created or updated as **global** (`isGlobal=true`) via the v4 API, reject it if any concretely-referenced user or group is **locally managed**" — [[sources/eng-910350-index]]
