# ADR-0012 — Trivial Composition

**Status:** Accepted  
**Date:** 2026-10-24  
**Revised:** 2026-10-25: screens follow the amended ADR-0009; every capability has a one-page host contract  
**Accepted:** 2026-10-25, with the host contract's limit raised from 150 to 200 lines  
**Amended:** 2026-10-31: replace the line ceiling with a concise, complete, independently usable consumer contract  
**Amended:** 2026-11-05: files join the shared shapes, filled by `storage`  
**Amended:** 2026-11-08: clarified provider-neutral composition and ownership boundaries  
**Amended:** 2026-11-12: ADR-0016-controlled extension remains subject to the composition budget  
**Amended:** 2026-11-13: capability responsibility is separate from implementation and deployment

> **In short:** composing a capability into an application must be trivial and measured. A host installs it, makes one call with what it already has (a database connection, the mail sender, the access decision) and writes no adapters of its own. Every capability fills its own ports by default, every capability asks for the same few things in the same shapes, and its tests prove a minimal host composes it within a fixed budget.

## Why

Applications should remain lightweight composition roots. Without a common contract for ports and composition, each application repeats store configuration, caller resolution, access-decision wiring and operational tasks. This makes capabilities harder to integrate and verify independently.

The cost is both code and coupling: consumers should compose a capability from its public contract without reading private implementation details.

## The decision

### 1. A composition budget

Composing a **Nuxt capability layer** into a Nuxt host takes:

- one package install, following current accepted `master` during development;
- one call in one server plugin, passing only what the host already owns;
- no more than **50 lines** of host code for that capability, counted as every line outside the host's own screens, content and data files that names the capability.

A framework-independent capability MUST offer one documented composition entry point using declared inputs, with no consumer-written stores, relays or adapters for its own rules. Its independent consumer example MUST record integration code size and justify any complexity; it does not require a Nitro plugin. Where that capability is integrated through a Nuxt adapter, the **same 50-line budget** applies to the complete capability-specific host integration, including its adapter setup. A different numerical limit for non-Nuxt consumers requires a separate accepted decision; none is established here.

What the host passes is limited to: services in the shared shapes of decision 2, connections and secrets ([ADR-0002](ADR-0002-composition-supplied-persistence-and-capability-owned-schemas.md)), configuration values, and decisions that belong to the application, such as which document types it registers. Everything else has a default.

### 2. The same shapes everywhere

Every capability asks for the shared needs in the same structural shapes, written here and copied verbatim into each contract. There is no shared runtime package: TypeScript's structural typing makes one provider fit every capability, and repository ownership does not couple to another ([ADR-0008](ADR-0008-independent-layer-delivery.md)).

| Need | Shape |
|---|---|
| Database | A `pg`-compatible pool, from which the capability opens its own connections and owns its schema (ADR-0002) |
| Caller | `resolveCaller(event) → { tenantId, subjectId \| null, assurance }` |
| Access | `decide(caller, action, resource) → { allowed, reason }`, failing closed |
| Mail | `send({ to, template, data })` |
| Events | The capability's outbox, drained by `relay()`; the host never reads its tables |
| Scheduled work | `tasks: { name, every, run }[]`, run by the host's one scheduler |
| Clock | `now()`, defaulting to the system clock |
| Files | The `StorageService` of `storage`'s contract: the host passes `storageFor('<consumer>')`; the consumer keeps each item's record and storage only the bytes. The consumer never imports `storage` |

### 2a. Provider-neutral security and operational composition

Shared shapes are **composition boundaries**, not ownership of another capability's detailed interface or implementation. A capability declares the services it needs, and the composition root connects providers through documented public contracts. Neither side imports another capability's private code, schema or infrastructure implementation.

Caller and access-decision providers must establish trustworthy server-side context and preserve the requesting capability's applicable assurance, tenant isolation and security requirements. No adapter may fabricate identity, elevate assurance, assume resource ownership or convert a missing, failed or ambiguous decision into permission. Domain invariants and any required independent approvals remain the responsibility of the capability that owns the operation.

A capability owns its business events, durable outbox and scheduled task handlers. Independently composable transport and execution capabilities may deliver published messages or run registered tasks through public interfaces, but never take ownership of the originating capability's business rules or persistence. A host must not reimplement those operations merely to compose capabilities.

The detailed meaning of caller fields, resource and decision types, task schedule syntax, execution coordination, message acknowledgements and delivery policies belongs to the responsible capability's `CONTRACT.md` and its own design. Where two capabilities must interoperate, their published contracts must be reconciled without introducing circular dependencies. Shared shapes change through an explicit architectural review under ADR-0008, with affected-consumer acceptance before integration is considered complete.

### 3. Capabilities fill their own ports

Each capability ships, in its own package:

- for capabilities that own PostgreSQL-persisted state, an implementation of their own stores, used when given a pool (stateless capabilities do not need a database);
- fakes of every port for tests, refused outside test configuration (ADR-0002, decision 2);
- a recording mail sender and caller resolver for tests and demonstrations.

Where capabilities belong together, one package wires them once and returns the shared shapes. `iam-integration` becomes that package for the IAM suite: given a pool and a mail sender, it returns `resolveCaller`, `decide` and `tasks`, ready for any capability, IAM or not.

### 4. Proven, not promised

Every **Nuxt capability layer** carries `examples/minimal-host`: a Nuxt application that composes it, and nothing else, within the 50-line budget. CI builds it, runs a smoke test and fails when its composition code exceeds the budget. A framework-independent capability instead carries a minimal independent consumer that proves its documented entry point, required inputs, contract sufficiency and failure behaviour, without pulling in Nuxt. If a Nuxt adapter is published, it also proves Nuxt composition within the 50-line budget. The platform test harness proves composed applications against the latest accepted development revisions. Acceptance results report the measured integration cost.

### 5. Screens

[ADR-0009](ADR-0009-headless-capability-layers.md), as amended on 2026-10-25, lets a capability ship its own screens as an optional presentation part. Using them costs the host only configuration: their paths, wording overrides, or `presentation: false`. That configuration counts against the budget; the screens themselves do not.

### 6. A self-contained consumer contract

Every capability, whether Nuxt-based or framework-independent, has a root `CONTRACT.md` containing the **shortest complete description of its public integration surface**. A consuming person or agent MUST be able to install, configure, compose and use the capability from that document alone, without reading other documentation or private source. There is no fixed line limit: omit repetition, internal design and implementation history, but never omit information required for correct use. Its sections, in this order (mark presentation-specific sections not applicable for packages without screens):

| Section | Holds |
|---|---|
| Install | The current development dependency and one configuration example, within the budget |
| Settings | Each environment variable or configuration value, its default, and what it does |
| Host passes | What the host must supply, in the shared shapes of decision 2, and nothing with a default |
| Client and server functions | Those screens and other capabilities call, with inputs, outputs and error codes |
| Permissions | Each permission it asks about, with its risk and effect |
| Screens | Its presentation part, if any: pages, paths, message keys, and how to switch it off |
| Headless surface | What any screen must still do (ADR-0009) |
| You will hit this | Rules a host would otherwise discover by failing: identifier formats, name patterns, what needs a second person or step-up |

Detailed internal specifications stay under `docs/` for maintainers and do not define a competing consumer interface. The public contract contains precise available imports, required port shapes, callable signatures, inputs, outputs, failure codes and relevant operational requirements; examples must not conceal required configuration or undefined dependencies. If the public contract, internal specification or implementation disagree, reconcile them against the accepted ADRs and normative capability requirements before acceptance. Implementation is evidence, not architectural authority. A handover to compose a capability names its `CONTRACT.md` and nothing else of the capability's.

### 7. Controlled specialisation does not bypass composition

[ADR-0016](ADR-0016-controlled-layer-specialisation-and-composition.md) permits documented public Nuxt extension surfaces and declarative archetypes. These are separate from capability-service port dependencies. A consumer that extends a presentation or composition layer still accounts for all host code needed to initialise its capabilities within the 50-line-per-capability budget, declares direct extension edges, and validates required providers, configuration and registration. No transitive inheritance silently satisfies a required service port, imports private code or transfers ownership. Existing completed layers require no migration solely because extension is permitted.

## Adopting it

Each capability implements the common contract and composition requirements in its own repository. Its own plan determines implementation order; consumers adopt the accepted public surface through protected pull requests and verify integration against current development dependencies (ADR-0008).

A suite's integration repository composes its members only after their independent public contracts are established. Capability-specific migration plans, metrics and implementation decisions stay with their owners.

## What follows

| Gains | Costs |
|---|---|
| Composing a capability is quick, cheap and predictable, for people and for agents | Stateful capabilities maintain their stores; every capability maintains appropriate test fakes and an example host |
| One access decision and one caller resolver serve every capability | The shared shapes must be agreed before the pilot, and changing one touches every contract |
| Applications hold product code, not plumbing | Consumer contracts require review for both completeness and unnecessary detail |
| Combinations proven in the harness carry straight into applications | Breaking changes require immediate consumer integration and corrections |

## Guardrails

- A capability's contract lists what a host must pass, and nothing in that list has a default the capability could supply itself.
- Every Nuxt capability layer's `examples/minimal-host` composes within the 50-line budget, or CI fails. A framework-independent capability must provide an independent consumer example; any Nuxt adapter must also meet that budget.
- No host writes a store, fake or relay for a capability that complies with this ADR; a review that finds one raises it against the capability.
- The shared shapes change only by amending this ADR.
- `CONTRACT.md` exists and keeps its required sections. Documentation checks validate structure and reference integrity, not an arbitrary line count.
- A minimal consumer integration or contract acceptance test MUST demonstrate that the contract is sufficient without reading the capability's other documents or private source; missing necessary information fails acceptance.
- Public API changes update `CONTRACT.md` in the same change. Consumers receive that document alone as their documentation handover.
