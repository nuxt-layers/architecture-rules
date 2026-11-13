# Capability Standard

**Status:** Normative

> **Summary:** Capabilities own clear responsibilities and publish contracts. Nuxt Layers and framework-independent packages are ways to implement them. Consumers use published interfaces, never private code.

This standard covers capability ownership, public contracts, repository rules, integration and testing. ADR-0001, ADR-0002, ADR-0008, ADR-0009, ADR-0011, ADR-0012, ADR-0015 and ADR-0016 govern its requirements.

## 1. Responsibility, implementation and deployment

A **capability** owns a cohesive responsibility and its business or platform rules. It need not be a Nuxt Layer or have a separate repository. A feature, component, endpoint or operation alone does not justify a new capability.

Choose independently:

| Decision | Meaning |
|---|---|
| Responsibility | What the capability does and owns |
| Implementation | How its functions are packaged and exposed |
| Deployment | Where its functions run |

The default deployment is a **modular monolith**. Separate services require an operational, scaling, security or ownership justification (ADR-0001).

### Classification

| Class | Responsibility | Examples |
|---|---|---|
| Foundation | Shared cross-cutting capabilities | Identity, Authentication, Authorisation |
| Platform | Reusable platform services | Publisher, Notifications |
| Domain | Business-specific functionality | Inventory, Bookings |
| Presentation | Screens and components consuming published contracts; no business-state ownership | `ui`, `content-editor`, `gallery` |

Classification describes responsibility, not technology. Presentation is a distinct classification of contribution, not a type of business-state owner.

### Implementation types

| Type | Requirements |
|---|---|
| Nuxt capability layer | Headless core and optional separable presentation under ADR-0009 |
| Framework-independent capability | Business or platform rules and public contracts without a Nuxt runtime requirement; may have a Nuxt adapter |
| Stateless library | Stateless reusable functions without stores, ports or screens (ADR-0011) |
| Infrastructure integration library | External infrastructure SDK and lifecycle integration, not business ownership (ADR-0015) |
| External application connector | Provider-specific integration behind an owning capability's documented interface |
| Application composition | Selects capabilities, supplies infrastructure and owns application routes and branding |

These forms are not mutually exclusive implementation components, but **not every library is a capability**. A capability may use a TypeScript library internally and expose its functions through a Nuxt Layer. Provider packages and external connectors do not acquire the consuming capability's ownership.

## 2. Ownership, dependencies and contracts

A capability owns its state, business rules and lifecycle. Repositories are separate only when ownership, reuse, testing, security or independent development warrant it.

A capability exposes deliberate public interfaces and keeps private implementation replaceable. Consumers must not import another capability's internal files, persistence adapters or database schemas.

Required and optional dependencies, including Nuxt extension edges, must be explicit, directional and acyclic. Optional dependencies have a documented absence mode; required dependencies fail closed when unavailable. A Nuxt extension does not grant ownership or private implementation access (ADR-0016).

A public contract may offer:

| Interface | Meaning |
|---|---|
| Service, command or query | Supported operations and reads |
| Data | Stable values crossing boundaries |
| Event | A fact published after an owned change, through the capability's outbox |
| Port | A dependency supplied through documented composition |
| Configuration | Validated settings and published extension points |
| HTTP API or typed client | Supported server or client integration |
| Presentation | Optional public screens, components and message catalogues |
| Errors | Stable codes and `messageKey` values, not display wording |

Contracts use domain language and documented export maps or entry points such as `/contracts`. They must not expose replaceable provider SDK types unless an accepted decision makes the provider part of the contract.

Validate untrusted inputs at runtime. Removing a public member, changing its meaning or shape, or narrowing its accepted inputs is breaking; additions also need compatibility review.

## 3. Repository and documentation

Use only the directories and files the implementation needs. A Nuxt capability layer may include:

```text
app/                      optional presentation
server/
shared/
tests/
docs/
examples/minimal-host/
nuxt.config.ts
capability.json
```

A framework-independent capability or library requires neither Nuxt files nor an artificial Nuxt host.

Independently developed packages retain applicable root documents:

| File | Purpose |
|---|---|
| `CONTRACT.md` | Concise, complete, independently usable public integration contract |
| `README.md` | Purpose and basic usage |
| `RESOURCES.md` | Verified external architectural references |
| `CHANGELOG.md` | Major changes |
| `PLAN.md` | Planned work while needed |
| `STORAGE-REGISTER.md` | Storage created or caused, when applicable |

Longer specifications and threat models belong under `docs/`. Security-critical packages maintain the required threat model and control evidence. Documentation follows ADR-0004 and ADR-0010.

Packages use the `@nuxt-layers/` scope where applicable and declare every imported dependency. Public entry points and compatibility requirements must be explicit. The repository's default branch is protected; material changes use pull requests.

## 4. Manifest and discoverability

Every independently developed **Nuxt capability layer** retains `capability.json` describing what it provides and requires:

```json
{
  "schemaVersion": "1",
  "name": "@nuxt-layers/example",
  "version": "0.0.0",
  "classification": "foundation",
  "provides": [{ "capability": "ExampleService", "contractVersion": "1" }],
  "requires": [{ "capability": "IdentityService", "contractVersion": "^1", "optional": false }]
}
```

The `requires` entries are contract dependencies, not Nuxt inheritance. Direct `extends` edges are declared separately as ADR-0016 requires. Metadata agrees with `CONTRACT.md` and package metadata but cannot replace or override them.

A framework-independent capability declares its provided contracts, dependencies and entry points in `CONTRACT.md` and package exports. **No new universal manifest schema is imposed here.** A future machine-readable capability registry needs separate schema approval; infrastructure integration libraries remain governed by ADR-0015 and do not need `capability.json`.

Manifest and contract versions describe interface compatibility and do not require release pins during development (ADR-0008).

## 5. Self-contained consumer contracts

Every independently developed capability has a root `CONTRACT.md` sufficient for a human or agent to install, configure and use it without reading internal source or other documents (ADR-0012 §6).

It records installation, settings, host-supplied services, public client and server functions, inputs, outputs, failure codes, permissions, applicable screens, headless presentation duties and relevant operational constraints. Mark non-applicable presentation sections accordingly; never hide essential configuration.

A Nuxt capability follows ADR-0012's shared structural port shapes for database, caller, access, mail, events, scheduled work, clock and files. Capability-specific port semantics belong to the owning `CONTRACT.md`; the standard does not invent replacements. Caller assurance, tenant isolation and failed-access behaviour must be preserved.

A contract change updates `CONTRACT.md` in the same change. Consumers receive only that contract as their integration handover.

## 6. Composition and specialisation

A Nuxt capability must satisfy ADR-0012: one package installation, one composition call in a server plugin and **at most 50 lines of capability-specific host code**. Host-supplied values remain limited to documented shared services, connections, secrets, configuration and application decisions. A published Nuxt adapter for a framework-independent capability also meets that budget.

A framework-independent capability provides one documented entry point and a minimal independent consumer example. That example reports integration cost and proves the contract without requiring Nuxt. No separate numeric budget for these consumers has been approved.

The host does not implement a capability's stores, fakes, relays or business adapters. Each capability supplies its own stores where needed and test-only fakes, as ADR-0012 requires.

Declarative specialisation may use validated, published extension points without depending on Nuxt. Nuxt `extends` is permitted only for documented public framework contributions, with declared merge and override rules and the combined acyclic dependency graph (ADR-0016). Neither form changes ownership or weakens security requirements.

## 7. Testing

A capability proves its published contract, business rules, dependency integration, security boundaries and consumption. Tests are appropriate to its implementation:

| Evidence | Proves |
|---|---|
| Contract and unit tests | Public behaviour and owned rules |
| Integration and compatibility tests | Dependencies, adapters and supported consumers |
| Security tests | Trust boundaries, permissions and fail-closed behaviour |
| Minimal consumer | Independent composition from `CONTRACT.md` |
| Nuxt minimal host | The 50-line budget, Nuxt integration and presentation controls |
| Application acceptance | Correct operation when capabilities are combined |

Nuxt capability layers retain `examples/minimal-host` and CI smoke testing under ADR-0012. A framework-independent capability without a Nuxt adapter uses an independent consumer instead. Intentional Nuxt `extends` requires tests of inherited behaviour, precedence, disabling, duplicate registration and dependency cycles under ADR-0016.

## 8. Development integration and change

Check the [Capability Catalogue](catalogue.md) before building overlapping functionality. Read a dependency's root `CONTRACT.md`; do not inspect or copy its private implementation to integrate it.

During active development, dependencies use the latest accepted `master` under ADR-0008. Workspaces may help locally, but acceptance uses current upstream revisions. Committed lockfiles record tested resolution without freezing development to stale revisions. Consumers test compatibility and fix forward.

Package releases remain suspended pending the approved release lifecycle. Contract changes are documented and tested, including breaking changes. Completed capabilities are not repackaged or migrated solely because another implementation form is now permitted.
