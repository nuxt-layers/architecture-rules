# ADR-0001 — Capability Repositories and Thin Composition Shells

**Status:** Accepted  
**Date:** 2026-09-26  
**Amended:** 2026-10-26: brought in line with ADR-0010 and later decisions

> **In short:** capabilities live in their own repositories when independence earns it, and applications stay thin composition roots that choose versions, pass infrastructure and own routes and branding. Layers meet only through public contracts; the default runtime is one modular monolith.

## Why

The ecosystem is expected to host multiple Nuxt capabilities, including shared UI, authentication, identity, authorisation, privacy, dashboard/workspace functionality and independently meaningful applications such as Architecture Registry.

Keeping every capability in one host repository would make independent reuse and versioning harder. At the other extreme, giving every component its own repository would create unnecessary maintenance and dependency overhead.

## The decision

The `nuxt-layers` organisation will host separately versioned repositories for cohesive bounded capabilities where independent development provides material value.

Nuxt applications will remain lightweight composition roots. They select compatible capability versions, provide application and deployment configuration, connect infrastructure and own application-specific routing and branding.

Cross-layer interaction occurs through declared public contracts. Repository separation does not permit consumers to depend upon another capability's private implementation.

The default runtime remains a modular monolith. A separate repository does not mean that the capability must run as a separate service.

## What follows

| Gains | Costs |
|---|---|
| Capability reuse across multiple Nuxt applications | Additional repository/release administration |
| Independent testing and release | Explicit version compatibility management |
| Explicit compatibility boundaries | Cross-repository changes may require coordinated releases |
| Clearer ownership and encapsulation | Contract design becomes a mandatory engineering discipline |
| Infrastructure and host applications can evolve independently |  |

## Guardrails

A new repository requires a bounded-capability justification. Microscopic repositories are discouraged.

The dependency graph must remain acyclic, and capabilities are connected through their ports by the host's composition call ([ADR-0012](ADR-0012-trivial-composition.md)), never by mutual domain dependencies.
