# ADR-0015 — Infrastructure Integration Libraries

**Status:** Accepted  
**Date:** 2026-11-10

> **In short:** a framework-independent infrastructure integration library may centralise a shared external system's client and worker integration without becoming a Nuxt Layer, a business capability or a network service. Its consumers retain their own rules, state and ports.

## Why

ADR-0011 permits small stateless libraries such as `rich-text`, while the Layer Standard describes Nuxt capability layers and those libraries. Some infrastructure integrations need more than stateless formatting functions: connection lifecycles, validated credentials, external SDK adaptation and independently deployed worker support. Duplicating these concerns in Scheduler, Workflow and other capabilities would create inconsistent integrations.

The proposed `temporal-gateway` is such an integration. It is a reusable TypeScript package wrapping selected Temporal SDK operations, not a new business capability. Treating it as a Nuxt Layer would incorrectly require a Nuxt entry point and capability manifest; treating it as an ADR-0011 stateless library would misdescribe its lifecycle responsibilities.

## The decision

1. **Distinct package kind.** An **infrastructure integration library** is a framework-independent package that encapsulates technical integration with one external infrastructure system. It is neither a Nuxt Layer nor the stateless-format library defined by ADR-0011. It may manage client connections, SDK adaptation, operational diagnostics and worker lifecycle, but must not own business policies, domain state or business persistence.
2. **Explicit boundaries.** Capabilities retain their public contracts, business decisions, event outboxes, scheduled task handlers and any capability-owned persistence. A gateway does not create a cross-capability business registry, orchestrate business processes, provide generic service routes or become an application-level network gateway.
3. **Composition and dependencies.** Hosts or trusted server-side composition code provide infrastructure configuration and credentials. Capabilities can use a shared integration library behind their published interfaces without requiring consumers to import the library. Dependency direction remains acyclic; an integration library never imports a consuming capability's implementation. Provider-neutral structural ports from ADR-0012 remain unchanged and do not become mandatory imports.
4. **Package contract.** The library exposes deliberate compiled package exports and a standalone root `CONTRACT.md` sufficient for an independent Node.js consumer. It documents configuration, lifecycle and ownership, public functions and errors, compatibility, security and worker deployment where applicable. It must not require Nuxt, Nitro, Vue, a `nuxt.config.ts` or a `capability.json`. Its own package metadata and contract define its integration surface; it does not claim a Foundation, Platform or Domain capability classification.
5. **Security and operations.** Validate untrusted configuration at runtime. Use trusted credentials and explicit infrastructure scopes; do not infer tenant authorisation from namespaces, task queues or other infrastructure identifiers. Keep credentials and confidential payloads out of diagnostics. The host owns provider selection, deployment and infrastructure persistence; a library may expose health and readiness but does not become the platform's logging or observability authority.
6. **Evidence and development.** Use the current accepted development `master` under ADR-0008, protected-branch pull requests and independent Node.js contract/integration/security tests. Test real provider operations and documented runtime compatibility where applicable. Record created or caused persistent storage in `STORAGE-REGISTER.md` when required by Privacy §5; a library with no storage of its own must not present the external provider's database as its own schema.

## Adopting it

The Layer Standard and Architecture overview distinguish infrastructure integration libraries from Nuxt capability layers and ADR-0011 stateless libraries. The Capability Catalogue lists integration libraries separately from capabilities. The `temporal-gateway` repository implements the first instance of this kind; its own `CONTRACT.md` sets the detailed Temporal API, not this ADR.

## What follows

| Gains | Costs |
|---|---|
| One maintained integration for multiple independently owned capabilities | A shared infrastructure dependency must be versioned and tested against its consumers |
| No artificial Nuxt runtime or capability metadata for Node-only packages | Distinct documentation and test expectations for infrastructure libraries |
| Business ownership and ADR-0012 shared ports remain intact | Provider-specific failure modes and runtime compatibility need explicit acceptance |

## Guardrails

- No capability may delegate its business rules, authorisation or owned persistence to an infrastructure integration library.
- A library's own API is not an amendment to ADR-0012 and does not redefine its shared structural ports.
- The ADR-0011 stateless-library category remains unchanged.
- The library is not a separately deployed service by default; workers and the external infrastructure may be deployed separately.
- Additional infrastructure libraries need a specific reuse and ownership justification, not merely convenience or a new SDK wrapper.
