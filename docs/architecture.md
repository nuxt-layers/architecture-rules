# Architecture

> **Summary:** capabilities own defined responsibilities and publish contracts; Nuxt Layers and framework-independent packages may implement them. Applications compose capabilities without access to private implementation. Dependencies run one way, security stays on the server, and the default is a modular monolith. See the [Capability Standard](capability-standard.md).

**Status:** Normative

## 1. The model

| Part | Does |
|---|---|
| **Capabilities** | Own bounded responsibilities, state and rules through published contracts; implementation and deployment are separate decisions (ADR-0001). Classified Foundation, Platform or Domain |
| **Nuxt Layers and presentation** | Integrate capability services into Nuxt, with optional separable screens where applicable (ADR-0009); presentation contributions own no domain state |
| **Framework-independent capabilities** | Implement public business or platform contracts without requiring Nuxt; may expose a Nuxt adapter (ADR-0001, ADR-0012) |
| **Stateless libraries** | Provide reusable stateless functionality such as `rich-text` without owning a capability (ADR-0011) |
| **Infrastructure integration libraries** | Framework-independent packages for shared external-system integration, such as `temporal-gateway`; not Nuxt Layers or business capabilities ([ADR-0015](decisions/ADR-0015-infrastructure-integration-libraries.md)) |
| **Suites** | Capabilities designed together, such as the IAM suite, wired once by their integration package ([ADR-0005](decisions/ADR-0005-iam-suite.md), [ADR-0012](decisions/ADR-0012-trivial-composition.md)) |
| **Applications** | Composition roots: select the latest accepted development revisions, pass connections, secrets, configuration and application decisions, set routes and branding, and deploy. Domain behaviour never defaults to the application |

## 2. Invariants

1. A capability is defined by its bounded responsibility and public contract, not by its framework, repository or deployment. Stateless libraries and infrastructure integration libraries remain distinct (ADR-0001, ADR-0011, ADR-0015).
2. Capabilities interact only through declared public contracts, ports and events, never another capability's private code, storage or schema.
3. Dependencies form a directed acyclic graph, counting both public-contract dependencies and direct Nuxt extension edges. If two capabilities seem to need each other, move the shared contract to a boundary, use an event or port, or redraw the boundary (ADR-0016).
4. A dependency targets a capability's contract, not how it is built or deployed: a capability needing access decisions asks for the shared `decide` shape, and the deciding capability learns nothing of its internals.
5. Prefer contract-based composition to inheritance. A layer MAY deliberately specialise a published extension surface or use Nuxt `extends` for documented configuration/presentation/composition reuse under [ADR-0016](decisions/ADR-0016-controlled-layer-specialisation-and-composition.md). This never grants access to private capability internals or transfers domain ownership. The host still composes capability ports within ADR-0012's budget.
6. Authentication, Identity and Authorisation are separate capabilities, even where one package wires them together. A person may belong to any number of groups, and access can express ownership by a person or group, roles and permissions, and resource-specific rules.
7. Access is decided on the server. Client-side checks are for the experience only.
8. Every persistent resource needing isolation has explicit ownership and access rules.
9. Infrastructure is replaceable through ports where practical. Capabilities depend on their own stores, not a hosting vendor's API. PostgreSQL is the preferred relational database; row-level security may add protection but never replaces application authorisation.
10. Security, privacy, accessibility and internationalisation are designed in from the start, never added late.
11. A capability declares what it provides and requires, with optional dependencies marked and behaviour without them documented. Nuxt layers retain `capability.json`; other packages document their contracts and exports without an imposed universal manifest (Capability Standard §4). Deliberate Nuxt extension edges are separately declared and tested under ADR-0016. Failures are documented codes; internal exceptions never cross a boundary.

## 3. Deployment

The default is a modular monolith: capability separation is logical and contractual, not network distribution. A capability becomes a separate service only when operations, scale, security or ownership justify the cost of distribution.

## 4. Routes and presentation

A capability's HTTP API and protocol endpoints sit under its own namespace, and the host may switch the API off. Its presentation part's pages can be moved or switched off; every other page belongs to the application, which may separate public pages from administration, for example on different hosts.

Presentation meets WCAG 2.2 AA as a minimum, supports internationalisation even where one locale is enabled, uses semantic HTML and Theme Manager's vocabulary, and provides Schema.org structured data from authoritative data where public. People's preferences may change colour, type, density and layout, never below the accessibility baseline (see the [presentation, internationalisation and accessibility baseline](presentation.md)).

## 5. Security and privacy

The minimum is OWASP ASVS 5.0 Level 2 across the platform, with relevant Level 3 requirements for security-sensitive capabilities and operations. NIST CSF 2.0 structures the security programme, NIST SP 800-218 guides secure development and NIST SP 800-63-4 guides identity assurance. Controls are layered across browser, server, storage, supply chain and deployment. Cost never silently weakens a required control: a deferred control is recorded with its risk treatment ([Security](security.md)).

Privacy follows data protection by design: storage and access technologies, telemetry and third-party integrations are classified by purpose, never introduced implicitly ([Privacy Architecture](privacy.md)).

## 6. Change

A change to an invariant, the contract model, a dependency rule or an ecosystem-wide standard needs an ADR or a revision of the governing document. Compatibility and migration are always explicit; during development consumers integrate the latest accepted `master` under ADR-0008.
