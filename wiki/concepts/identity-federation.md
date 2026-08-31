---
type: concept
created: 2026-08-30
tags: [method]
aliases: [externally-managed identities, LOCAL user, SERVICE_ACCOUNT]
sources:
  - "[[sources/eng-910350-index]]"
---

# identity federation

## Definition
The rule that global, lattice-replicated ACPs may only grant to identities that exist across clusters. Local users and service accounts are cluster-scoped and have no stable cross-cluster identity.

## Key Characteristics
- Allowed: SAML / LDAP / external IDP users, NC-propagated groups
- Rejected on global ACP create/update: LOCAL user, SERVICE_ACCOUNT
- Complements enrich→resolve UUID rewrite on the replica path

## Applications
- ENG-910350 authoring-side validator on NC (leader)
- ENG-910347 resolve-on-target rewrite so a joining PC maps foreign UUIDs to local ones

## Related Concepts
- [[concepts/global-acp|global ACP]]
- [[concepts/shard-copy|shard copy]]
- [[concepts/lattice|lattice]]

## Related Entities
- [[entities/eng-910350|ENG-910350]]
- [[entities/eng-910347|ENG-910347]]

## Mentions in Source
- "Only externally-managed identities (SAML / LDAP / external IDP, and NC-propagated groups) may appear on a global, lattice-replicated ACP." — [[sources/eng-910350-index]]
- "Local users / service accounts are cluster-scoped and have no stable cross-cluster identity" — [[sources/eng-910350-index]]
