---
type: entity
created: 2026-08-30
tags: [product]
aliases: [Nutanix hypervisor]
sources:
  - "[[sources/nutanix-technical-stack]]"
---

# AHV

## Description
Nutanix hypervisor layer. VMs and Kubernetes sit above it. Explicitly not an allowed `products` value for the IAM products field.

## Related Entities
- [[entities/nkp|NKP]]
- [[entities/eng-932537|ENG-932537]]

## Related Concepts
- [[concepts/products-field|products]]

## Mentions in Source
- "This all runs on the nutanix hypervisor layer(AHV)" — [[sources/nutanix-technical-stack]]
- "Explicitly NOT allowed: `AHV`, `CALM`, `UNKNOWN`, `Xi`." — [[sources/eng-932537-overview]]
