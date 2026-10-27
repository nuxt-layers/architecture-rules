# Capability Catalogue

> **In short:** the capabilities that have a repository, what each owns, and the rules every capability follows. Each entry is one line; a capability's own repository holds its detail ([ADR-0004](decisions/ADR-0004-documentation-placement.md)). Candidates are listed by name at the end; the order of work belongs to each capability's and application's own plan.

## Governing rules

1. Each layer implements one cohesive bounded capability, classified as Foundation, Platform or Domain, with a headless core and an optional presentation part ([ADR-0009](decisions/ADR-0009-headless-capability-layers.md)).
2. Applications are thin composition roots, supplying infrastructure, configuration, routing and deployment choices ([ADR-0001](decisions/ADR-0001-capability-repositories-and-composition-shells.md), [ADR-0012](decisions/ADR-0012-trivial-composition.md)).
3. Dependencies are directed, acyclic, explicit and contract-based. Public contracts are exposed through deliberate entry points such as `/contracts`, and independently versioned layers declare capability metadata in the manifest standard.
4. Stateful capabilities own their persistence ports, schemas, migrations and any other stores ([ADR-0002](decisions/ADR-0002-composition-supplied-persistence-and-capability-owned-schemas.md), [ADR-0006](decisions/ADR-0006-polyglot-persistence-and-data-store-security.md)). No shared persistence service is permitted, and required persistence fails closed when its provider is absent.
5. Identity owns users, groups, the single-parent group hierarchy and memberships; Profile owns personal data; Authentication owns sessions; Authorisation owns access decisions. Every human identity has one system-managed personal group.
6. Every protected operation is authorised on the server. Security follows OWASP ASVS 5.0 Level 2, with applicable Level 3 controls for security-sensitive operations.
7. Presentation meets WCAG 2.2 AA, supports internationalisation and uses Theme Manager's semantic vocabulary. Privacy, data minimisation, retention and consent apply across capabilities.
8. Separate deployment is optional; the default architecture is a modular monolith.

## Capabilities

| Capability | Class | Owns |
|---|---|---|
| `theme-manager` | Foundation | Theme definitions, the semantic token vocabulary, contrast checks, selection and preferences; the only authority for the effective theme |
| `configuration` | Foundation | The facts a site, tenant or group states about itself, changed through approvals |
| `identity` | Foundation (IAM) | Identities, each person's personal group, groups, the single-parent hierarchy, tenants and memberships with their lifecycle; no personal data beyond opaque identifiers |
| `authentication` | Foundation (IAM) | Credential verification, sign-in identifiers, sessions, rotation, revocation and reauthentication |
| `authorisation` | Foundation (IAM) | Server-side, resource-aware access decisions, with no implicit inheritance through the group hierarchy |
| `profile` | Foundation (IAM) | The personal data describing a person, keyed by an Identity identifier: what each context may see, under the person's control, and their access, correction, export and erasure requests |
| `iam-integration` | Foundation (IAM) | The IAM suite's architecture, cross-capability processes and the adapters connecting its members ([ADR-0005](decisions/ADR-0005-iam-suite.md)); no data of its own |
| `rich-text` | Library | The one safe formatted-text format, its profiles, checks and safe tree; never renders ([ADR-0011](decisions/ADR-0011-shared-rich-text-and-configurable-copy.md)) |
| `publisher` | Platform | Documents of host-registered types: drafts, versions, approval, scheduling, addresses, listings and feeds |
| `media` | Platform | Images: web-ready variants, metadata removal, alt text, group ownership and allowances; the files themselves in a host-supplied bucket |
| `ui` | Presentation | Generic, fully styled components on Theme Manager's vocabulary; knows no capability |
| `content-editor` | Presentation | Editing screens for Publisher's documents, saving through Publisher's functions; no data of its own |
| `gallery` | Presentation | Albums and photo of the week as Publisher types over media's images; no data of its own |

## Candidates

Possible capabilities, not planned: each is built only when an application needs it and its bounded responsibility is established (below). Their definitions are in [Resources](../RESOURCES.md): *Catalogue v0.4*.

- **Foundation:** `privacy`, `logging-service`, `uuidv7-generator`, `caching-service`, `clock-service`.
- **Platform:** `observability-service`, `notification-service`, `event-bus-service`, `job-service`, `scheduling-service`, `workflow-service`, `process-service`, `task-service`, `project-service`, `reporting-service`, `search-service`, `document-service`, `location-service`, `tenancy-service`.
- **Domain:** `organisation-service`, `company-service`, `department-service`, `club-service`, `committee-service`, `membership-service`, `office-service`; `product-service`, `pricing-service`, `inventory-service`, `order-service`, `invoice-service`, `billing-service`, `accounting-service`, `payment-service`, `subscription-service`, `shop-service`, `marketplace-service`; `warehouse-service`, `logistics-service`, `asset-service`, `workshop-service`, `booking-service`; `course-service`, `enrolment-service`, `training-service`, `event-service`, `game-service`.
- **Presentation and composition:** `user-management-ui`, `warehouse-management-ui`, `workflow-management-ui`, `workshop-management-ui`, `company-console`, `integration-adapters`, `integrations`, `api-gateway`.

## Consolidations

- No shared `persistence-service` (ADR-0002). Generic `user-service` and `group-service` are Identity; sessions stay in Authentication.
- `blog-service` is a Publisher document type; `image-service`, `storage-service` and `media-library-ui` are `media`; `todo-manager` belongs in `task-service`.
- `iam-integration` is unrelated to the general-purpose `integrations` candidate.
- Existing repository names and contracts are kept unless an independently reviewed change justifies a migration.

## Starting a capability

A repository is created only once these are established:

1. purpose and bounded responsibility;
2. owned state, rules and lifecycle;
3. explicit exclusions;
4. provided public contracts and their versions;
5. required and optional capability contracts;
6. configuration and persistence-provider requirements;
7. security, privacy, accessibility and internationalisation obligations;
8. independent contract, integration and consumer acceptance tests.

The group model is defined in [Identity and Access](identity.md) ([ADR-0003](decisions/ADR-0003-group-model-and-identity-first.md)).
