# ADR-0003 — Group Model Invariants

**Status:** Accepted  
**Date:** 2026-10-08  
**Amended:** 2026-10-26: brought in line with ADR-0010 and later decisions

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
- Tenant provisioning and lifecycle remain a candidate `tenancy-service`; until one exists, tenants are supplied through Identity and the composition root.
