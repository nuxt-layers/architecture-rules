# Architecture

> **In short:** the ecosystem is bounded capabilities, built as Nuxt Layers, composed into light applications. Each capability owns its data and rules and offers a public contract; applications choose capabilities, pass what they already own and draw what is theirs. Dependencies run one way, security is enforced on the server, and the default is one deployable application. What a layer is and how it is built is in the [Layer Standard](layer-standard.md).

**Status:** Normative

## 1. The model

| Part | Does |
|---|---|
| **Capabilities** | Own one bounded responsibility each, with a headless core and an optional presentation part ([ADR-0009](decisions/ADR-0009-headless-capability-layers.md)). Classified Foundation, Platform or Domain |
| **Presentation layers and libraries** | Draw screens over capabilities' contracts, or provide stateless shared formats such as `rich-text` |
| **Suites** | Capabilities designed together, such as the IAM suite, wired once by their integration package ([ADR-0005](decisions/ADR-0005-iam-suite.md), [ADR-0012](decisions/ADR-0012-trivial-composition.md)) |
| **Applications** | Composition roots: choose capability versions, pass connections, secrets, configuration and application decisions, set routes and branding, and deploy. Domain behaviour never defaults to the application |

## 2. Invariants

1. A layer is a bounded capability, not a convenient collection of files.
2. Layers interact only through declared public contracts, ports and events, never another layer's private code, storage or schema.
3. Dependencies form a directed acyclic graph. If two capabilities seem to need each other, move the shared contract to a boundary, use an event or port, or redraw the boundary.
4. A dependency targets a capability's contract, not how it is built or deployed: a capability needing access decisions asks for the shared `decide` shape, and the deciding capability learns nothing of its internals.
5. Composition, not inheritance: Nuxt `extends` is a mechanism, never a chain of layers extending layers. The host combines peer capabilities, each filling its own ports by default; hosts write no adapters (ADR-0012).
6. Authentication, Identity and Authorisation are separate capabilities, even where one package wires them together. A person may belong to any number of groups, and access can express ownership by a person or group, roles and permissions, and resource-specific rules.
7. Access is decided on the server. Client-side checks are for the experience only.
8. Every persistent resource needing isolation has explicit ownership and access rules.
9. Infrastructure is replaceable through ports where practical. Capabilities depend on their own stores, not a hosting vendor's API. PostgreSQL is the preferred relational database; row-level security may add protection but never replaces application authorisation.
10. Security, privacy, accessibility and internationalisation are designed in from the start, never added late.
11. A capability declares what it provides and requires, with optional dependencies marked and behaviour without them documented. Its failures are documented codes; internal exceptions never cross a boundary.

## 3. Deployment

The default is a modular monolith: capability separation is logical and contractual, not network distribution. A capability becomes a separate service only when operations, scale, security or ownership justify the cost of distribution.

## 4. Routes and presentation

A capability's HTTP API and protocol endpoints sit under its own namespace, and the host may switch the API off. Its presentation part's pages can be moved or switched off; every other page belongs to the application, which may separate public pages from administration, for example on different hosts.

Presentation meets WCAG 2.2 AA as a minimum, supports internationalisation even where one locale is enabled, uses semantic HTML and Theme Manager's vocabulary, and provides Schema.org structured data from authoritative data where public. People's preferences may change colour, type, density and layout, never below the accessibility baseline (see the [presentation, internationalisation and accessibility baseline](presentation.md)).

## 5. Security and privacy

The minimum is OWASP ASVS 5.0 Level 2 across the platform, with relevant Level 3 requirements for security-sensitive capabilities and operations. NIST CSF 2.0 structures the security programme, NIST SP 800-218 guides secure development and NIST SP 800-63-4 guides identity assurance. Controls are layered across browser, server, storage, supply chain and deployment. Cost never silently weakens a required control: a deferred control is recorded with its risk treatment ([Security](security.md)).

Privacy follows data protection by design: storage and access technologies, telemetry and third-party integrations are classified by purpose, never introduced implicitly ([Privacy Architecture](privacy.md)).

## 6. Change

A change to an invariant, the contract model, a dependency rule or an ecosystem-wide standard needs an ADR or a revision of the governing document. Compatibility and migration are always explicit.
