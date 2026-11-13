# ADR-0001 — Capability Repositories and Thin Composition Shells

**Status:** Accepted  
**Date:** 2026-09-26  
**Amended:** 2026-10-26: brought in line with ADR-0010 and later decisions  
**Amended:** 2026-11-12: ADR-0016 permits documented Nuxt extension without changing service composition  
**Amended:** 2026-11-13: capability responsibility is separate from implementation and deployment

> **In short:** capabilities live in their own repositories when independence earns it, and applications stay thin composition roots that select capabilities, pass infrastructure and own routes and branding. Layers meet only through public contracts; the default runtime is one modular monolith.

## Why

The ecosystem is expected to host multiple Nuxt capabilities, including shared UI, authentication, identity, authorisation, privacy, dashboard/workspace functionality and independently meaningful applications such as Architecture Registry.

Keeping every capability in one host repository would make independent reuse and versioning harder. At the other extreme, giving every component its own repository would create unnecessary maintenance and dependency overhead.

## The decision

A **capability** is defined by the responsibility and rules it owns and the public contract it offers, not by its framework, package or deployment. A user-facing feature need not be a separate capability or repository. Responsibility, implementation and deployment are separate decisions.

A capability MAY use a Nuxt Layer, a framework-independent TypeScript implementation with a Nuxt adapter, or another justified form. A Nuxt Layer remains a framework integration mechanism, not the definition of a capability. Stateless libraries (ADR-0011) and infrastructure integration libraries (ADR-0015) remain distinct: using TypeScript alone does not make a package a capability.

The `nuxt-layers` organisation will host separate repositories for cohesive bounded capabilities where independent development provides material value.

Nuxt applications will remain lightweight composition roots. They integrate the latest accepted development revisions of capabilities, provide application and deployment configuration, connect infrastructure and own application-specific routing and branding.

Cross-capability interaction occurs through declared public contracts. Repository separation does not permit consumers to depend upon another capability's private implementation.

The default runtime remains a modular monolith. A separate repository does not mean that the capability must run as a separate service.

## What follows

| Gains | Costs |
|---|---|
| Capability reuse across multiple Nuxt applications | Additional repository and integration administration |
| Independent ownership and testing with continuous integration | Cross-repository acceptance testing |
| Explicit compatibility boundaries | Cross-repository changes may require consumer corrections |
| Clearer ownership and encapsulation | Contract design becomes a mandatory engineering discipline |
| Infrastructure and host applications can evolve independently |  |

## Guardrails

A new repository requires a bounded-capability justification. Microscopic repositories are discouraged.

The combined dependency graph (including contract requirements and direct Nuxt extension edges) must remain acyclic. Capability services are connected through their public ports by the host's composition call ([ADR-0012](ADR-0012-trivial-composition.md)), never by mutual domain dependencies. Deliberate Nuxt `extends` for documented public framework contributions and schema-validated specialisation are permitted under [ADR-0016](ADR-0016-controlled-layer-specialisation-and-composition.md); this does not grant access to private implementation or alter domain ownership.
