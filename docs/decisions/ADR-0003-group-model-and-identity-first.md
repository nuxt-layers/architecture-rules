# ADR-0003 — Group Model Invariants

**Status:** Accepted  
**Date:** 2026-10-08  
**Amended:** 2026-10-26: brought in line with ADR-0010 and later decisions  
**Supplemented:** 2026-11-16: ADR-0017 separately establishes tenant parentage and digital tenant governance; group invariants remain unchanged

> **In short:** the group model's invariants are normative across capabilities that use identities, groups and memberships.

## Why

The [group model](../identity.md) was added as a proposed extension to the Identity and Authorisation architecture. It defines personal (unary) groups, single-parent group hierarchies, membership lifecycle, the separation of ownership from creator provenance and of tenants from groups, the absence of implicit hierarchy inheritance, and revocation on departure. It required an accepted ADR before its invariants became normative. It is now part of [Identity and Access](../identity.md).

The group model is a shared boundary; capability-specific implementation and development order belong to the owning repositories.

## The decision

1. The group model is **accepted as normative architecture**. Its invariants and acceptance criteria ([Identity and Access](../identity.md) §3 to §11) apply to Identity, Authorisation and any capability that associates domain entities with groups.
2. Identity owns the group and membership model; other capabilities consume the public contract and must not duplicate Identity's authority.
3. Individual capability plans, rather than this ADR, determine development sequence.

## What follows

| Gains | Costs |
|---|---|
| Identity is built against settled invariants rather than a proposal | Identity and Authorisation must meet the group model's acceptance tests, including membership-derived revocation and cache invalidation |
| Shared group semantics prevent competing interpretations of membership and access | Each capability must prove conformance at its own boundary |

## Guardrails

- Changes to the group model's invariants require a new ADR.
- Tenant records, provenance and governance are owned by Identity; the host/composition root coordinates secure provisioning through documented public boundaries. [ADR-0017](ADR-0017-tenant-hierarchy-and-accountable-digital-control.md) adds a separately scoped tenant hierarchy and accountable tenant digital ownership, **not** a second group forest or an independently approved `tenancy-service`. The accepted group founding-owner, hierarchy and membership invariants remain unchanged.
