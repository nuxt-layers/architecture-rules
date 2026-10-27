# ADR-0003 — Group Model and Identity Before Logging

**Status:** Accepted  
**Date:** 2026-10-08  
**Amended:** 2026-10-26: brought in line with ADR-0010 and later decisions

> **In short:** the group model became normative, and Identity was built before Logging Service because Authorisation and Authentication needed real groups and memberships.

## Why

The [group model](../identity.md) was added as a proposed extension to the Identity and Authorisation architecture. It defines personal (unary) groups, single-parent group hierarchies, membership lifecycle, the separation of ownership from creator provenance and of tenants from groups, the absence of implicit hierarchy inheritance, and revocation on departure. It required an accepted ADR before its invariants became normative. It is now part of [Identity and Access](../identity.md).

The Authorisation capability (`nuxt-layers/authorisation`, contract version 2) has already been reconciled with that definition: hierarchy inheritance is opt-in per role assignment, tenant isolation is a separate check, creator provenance grants nothing, only active memberships count, and its directory port carries a revocation consistency guarantee.

The capability catalogue v0.3 ([Resources](../../RESOURCES.md): *Catalogue v0.3*) §10 placed Logging Service next. Authorisation's persistence and server functions, and Authentication's principal-to-actor integration, cannot be completed against a real implementation until Identity provides users, groups, hierarchy and memberships.

## The decision

1. The group model is **accepted as normative architecture**. Its invariants and acceptance criteria ([Identity and Access](../identity.md) §3 to §11) apply to Identity, Authorisation and any capability that associates domain entities with groups.
2. **Identity** is the next capability to be developed, before Logging Service. Identity implements the group model; Authorisation consumes it through its directory port.
3. Logging Service follows Identity as the next foundational reference capability.

## What follows

| Gains | Costs |
|---|---|
| Identity is built against settled invariants rather than a proposal | Identity and Authorisation must meet the group model's acceptance tests, including membership-derived revocation and cache invalidation |
| Authorisation's phase 2 and Authentication's host integration are unblocked as soon as Identity's contract exists | Logging Service, and the structured audit sinks that would consume capabilities' events, are deferred; capabilities continue to emit events to host-supplied sinks |
| The catalogue's development order matches the order in which capabilities are actually needed |  |

## Guardrails

- Changes to the group model's invariants require a new ADR.
- Tenant provisioning and lifecycle remain a candidate `tenancy-service`; until one exists, tenants are supplied through Identity and the composition root.
