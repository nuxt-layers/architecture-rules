# Capability Catalogue

> **In short:** authoritative ownership boundaries for established capabilities and provisional boundaries for candidate capabilities. The catalogue defines vocabulary and intended relationships, not delivery status or implementation plans. Capability detail belongs in its own repository ([ADR-0004](decisions/ADR-0004-documentation-placement.md)).

## Governing rules

1. Each layer implements one cohesive bounded capability, classified as Foundation, Platform or Domain, with a headless core and an optional presentation part ([ADR-0009](decisions/ADR-0009-headless-capability-layers.md)).
2. Applications are thin composition roots, supplying infrastructure, configuration, routing and deployment choices ([ADR-0001](decisions/ADR-0001-capability-repositories-and-composition-shells.md), [ADR-0012](decisions/ADR-0012-trivial-composition.md)).
3. Contract dependencies and direct Nuxt extension edges are explicitly declared, directed and jointly acyclic. Service use is through public contracts and host-supplied ports; deliberate schema-validated archetype specialisation and Nuxt extension of documented public contributions are permitted under [proposed ADR-0016](decisions/ADR-0016-controlled-layer-specialisation-and-composition.md). No extension implies access to private source.
4. Stateful capabilities own their persistence ports, schemas, migrations and any other stores ([ADR-0002](decisions/ADR-0002-composition-supplied-persistence-and-capability-owned-schemas.md), [ADR-0006](decisions/ADR-0006-polyglot-persistence-and-data-store-security.md)). No shared persistence service is permitted, and required persistence fails closed when its provider is absent.
5. Identity owns users, groups, the single-parent group hierarchy and memberships; Profile owns personal data; Authentication owns sessions; Authorisation owns access decisions. Every human identity has one system-managed personal group.
6. Every protected operation is authorised on the server. Security follows OWASP ASVS 5.0 Level 2, with applicable Level 3 controls for security-sensitive operations.
7. Presentation meets WCAG 2.2 AA, supports internationalisation and uses Theme Manager's semantic vocabulary. Privacy, data minimisation, retention and consent apply across capabilities.
8. Separate deployment is optional; the default architecture is a modular monolith.

## Capabilities

| Capability | Class | Owns |
|---|---|---|
| `theme-manager` | Foundation | Theme definitions, the semantic token vocabulary, contrast checks, selection and preferences; the only authority for the effective theme |
| `clock` | Foundation | Canonical date/time values, parsing, formatting, temporal arithmetic, time zones and clock services |
| `configuration` | Foundation | The facts a site, tenant or group states about itself, changed through approvals |
| `identity` | Foundation (IAM) | Identities, each person's personal group, groups, the single-parent hierarchy, tenants and memberships with their lifecycle; no personal data beyond opaque identifiers |
| `authentication` | Foundation (IAM) | Credential verification, sign-in identifiers, sessions, rotation, revocation and reauthentication |
| `authorisation` | Foundation (IAM) | Server-side, resource-aware access decisions, with no implicit inheritance through the group hierarchy |
| `profile` | Foundation (IAM) | The personal data describing a person, keyed by an Identity identifier: what each context may see, under the person's control, and their access, correction, export and erasure requests |
| `iam-integration` | Foundation (IAM) | The IAM suite's architecture, cross-capability processes and the adapters connecting its members ([ADR-0005](decisions/ADR-0005-iam-suite.md)); no data of its own |
| `rich-text` | Library | The one safe formatted-text format, its profiles, checks and safe tree; never renders ([ADR-0011](decisions/ADR-0011-shared-rich-text-and-configurable-copy.md)) |
| `publisher` | Platform | Documents of host-registered types: drafts, versions, approval, scheduling, addresses, listings and feeds |
| `media` | Platform | Images: web-ready variants, metadata removal, alt text, group ownership and allowances; the files themselves through the Files shape ([ADR-0012](decisions/ADR-0012-trivial-composition.md)), which the host fills with `storage` |
| `storage` | Platform | Files' bytes in an object store behind one provider-agnostic port: keys under tenant and group, signed addresses, lifecycle, backups and usage figures; no records of its own |
| `ui` | Presentation | Generic, fully styled components on Theme Manager's vocabulary; knows no capability |
| `content-editor` | Presentation | Editing screens for Publisher's documents, saving through Publisher's functions; no data of its own |
| `gallery` | Presentation | Albums and photo of the week as Publisher types over media's images; no data of its own |
| `privacy` | Foundation | Consent and retention policy, which capabilities receive and apply |
| `legal-compliance` | Foundation | The legal duties that apply to each site by jurisdiction, such as UK GDPR and the Online Safety Act, and the constraints they impose beside Authorisation's decisions, such as legal holds that block deletion |
| `export` | Library | Turning a `rich-text` safe tree into files such as PDF and .docx, extensible by format |
| `scheduler` | Platform | Running every capability's `tasks` on time, like cron, for the host |
| `message-queue` | Platform | Carrying messages that something happened, such as "password changed", from capabilities' outboxes to the capabilities that listen; not calendar events, which are the `event-service` candidate |
| `workflow` | Platform | User-defined routes for review, sign-off and approval, per type and site; fills capabilities' approval ports |
| `content-security` | Platform | Scanning what people submit for dangerous content, links and abuse, on top of each layer's own input checks |
| `notifications` | Platform | Telling people something needs them, through their chosen channels; providers' keys come from the host's secret management |
| `comments-engine` | Platform | Comments on content and their moderation |
| `metadata` | Platform | Tags, categories, taxonomies and ontologies attached to content |
| `translate` | Platform | Translation of content and its workflow |
| `seo` | Platform | Redirects, sitemaps and search-engine metadata, driven by Publisher's address events |
| `site-hosting` | Platform | Where each site is served: custom domains, proving their ownership, and finding the site for a request's address; certificates and deployment stay with the host's proxy and provider (ADR-0014) |
| `integrations` | Platform | Adapters to outside services such as Google Docs, Microsoft 365 and social feeds |

## Infrastructure integration libraries

These are framework-independent packages, not independently composable Nuxt capabilities; their ownership and requirements follow [ADR-0015](decisions/ADR-0015-infrastructure-integration-libraries.md).

| Library | Owns |
|---|---|
| `temporal-gateway` | Shared Temporal connection, schedule, execution and worker SDK integration; no business scheduling, workflows, message transport or capability-owned persistence |

## Additional platform and architectural entries under consideration

The entries below describe additional bounded responsibilities without implying that an independent repository, a direct dependency, or a change to an established contract has been approved.

| Entry | Intended class | Scope and boundary |
|---|---|---|
| `architecture-rules` | Architecture governance (not a runtime layer) | Governs ADRs, normative standards and this catalogue; owns no application runtime data |
| `infrastructure` | Foundation candidate | Host/backend/deployment setup and operational configuration; infrastructure resources and domain data remain with the appropriate hosts/providers |
| `reviews` | Platform candidate | Customer ratings, testimonials, verified purchase/service evidence, moderation, merchant replies and trustworthy reputation aggregates; transaction and identity facts remain owned by source capabilities |
| `template-engine` | Library candidate | Secure deterministic template parsing, data binding and content rendering; no template persistence, document lifecycle, arbitrary code execution or business-data lookups |
| `organisation-model` | Domain / platform boundary to be decided | Versioned, schema-validated organisational archetypes, positions, non-authoritative reporting relationships and capability selection; cannot reown Identity groups/memberships or Authorisation decisions |

### Reviews and external reputation providers

`reviews` distinguishes verified purchase, verified service/experience, invited and unverified submissions. Verification requires evidence from the authoritative ordering, booking or equivalent source, not a self-asserted flag. Protect reviewer privacy, moderation auditability, business responses and transparent provenance. Reviews and reputational aggregation do not, by themselves, implement personalised product recommendations; a separate recommendation-algorithm boundary must be justified if needed.

The optional **Trustpilot adapter** belongs under `integrations` (conceptually `integrations/trustpilot`), not as a required `reviews` dependency or automatically as its own Nuxt Layer. It translates permitted invitations, reviews, ratings and replies through documented external APIs and retains source attribution and verification meaning. The reviews core must function without it; external provider terms, credentials and data transfers require independent approval.

### Organisational archetypes versus business domains

An organisational archetype MAY describe metadata schemas, positions, requested role profiles, typed reporting relationships and capability configuration. It MUST NOT become a second Identity group hierarchy, directly grant access, assert asset permissions, or absorb independent domain lifecycles. Permission profiles are requests interpreted under existing Authorisation and domain policies. Reporting relationships are not access-control inheritance.

`organisation`, `business`, `company`, `club`, `committee`, `department` and `office` remain candidates until independent state and lifecycle justify distinct domain owners; otherwise they may be archetypes, application compositions or specialised presentation over existing contracts. Legal-entity registration, committee decisions and other independently regulated lifecycles are examples that might justify distinct domains, not generic metadata fields.

### Relationship vocabulary

| Relationship | Meaning | Example (illustrative, not a declared dependency) |
|---|---|---|
| Requires | Essential public service/port contract; absence fails closed | Warehouse may require Inventory operations |
| Optionally uses | Optional published contract and documented absence mode | Reviews may integrate external reputation providers |
| Specialises | Validated archetype, configuration or documented extension surface | Greengrocer specialises a retail archetype |
| Extends (Nuxt) | Direct, documented Nuxt framework inheritance with merge/override rules | A retail app extends a presentation/composition shell |
| Composes | Application/integration assembly of independent capability contracts | Shop application combines Product, Pricing and Order |

These are distinct edges: `requires` is not synonymous with Nuxt `extends`, and a business event is not an inheritance mechanism. Dependency cycles are prohibited over their combined graph where edges denote runtime/build dependencies (ADR-0001 and proposed ADR-0016).

## Candidates

These names and descriptions are **design options**, not approved independent repositories, mandatory dependencies or delivery commitments. The candidate names follow the supplied CSV where meaningful; existing contracts and accepted ADRs take precedence.

### Foundation, platform and library candidates

| Candidate | Intended class | Potential bounded responsibility / exclusion |
|---|---|---|
| `audit` | Platform | Tamper-evident accountable business/security audit evidence and review; distinct from operational Logging and Observability |
| `caching` | Foundation | Cache provider ports, expiration and invalidation; source capabilities remain authoritative |
| `logging` | Foundation | Structured technical logs, correlation and sinks; not the authoritative business audit trail |
| `observability` | Platform | Metrics, traces, system health and diagnostics; not logging or business auditing |
| `uuidv7-generator` | Library candidate | UUIDv7 identifier generation; not ownership of entities or credentials, and not automatically a Nuxt Layer |
| `legal-compliance` | Foundation | Jurisdictional legal requirements and policy constraints, including holds; does not take over Authorisation |
| `privacy` | Foundation | Consent and retention policies and their enforcement contracts; private personal records stay with Profile |
| `api-gateway` | Composition / platform to decide | External API routing and exposure; not a replacement for capability APIs or server authorisation |
| `application-compositions` | Composition artefacts | Reusable application templates and selected capability wiring; generally not independent stateful services |
| `company-console` | Presentation | Cross-capability management screens; no business records |
| `user` | Presentation | Group-scoped user administration screens via Identity/Profile/Authorisation; no user records |
| `comments-engine` | Platform | Comments, threads, moderation and reply history on referenced resources |
| `content-security` | Platform | Content risk scanning and abuse screening; supplements rather than replaces capability validation |
| `document` | Platform candidate, unresolved | Distinct document-management requirement only if Publisher's document lifecycle is insufficient |
| `media` | Platform | Image handling and variants; file bytes remain the responsibility of Storage |
| `metadata` | Platform | Reusable tags, classification, taxonomies and associations; no ownership of annotated content |
| `search` | Platform | Indexing, query and relevance over authorised source records; no source-of-truth storage |
| `seo` | Platform | Sitemaps, redirects and search metadata from Publisher address facts |
| `translate` | Platform | Content translation and review; not application message localisation |
| `integrations` | Platform | External-provider adapters and synchronisation plumbing; domain meaning stays with owning capabilities |
| `location` | Platform | Coordinates, geospatial and place references; not all postal addresses or site tenancy |
| `reporting` | Platform | Authorised report definitions, aggregation and output; no source transaction records |
| `site-hosting` | Platform | Domain verification and site lookup; certificate/hosting provider ownership stays with the host |
| `notifications` | Platform | User-facing notification routing, channel selection and delivery; no external-provider secrets |
| `process` | Platform candidate | Business process definitions and outcomes; execution uses Workflow/Scheduler without duplicating their responsibilities |
| `project` | Platform candidate | Coordinated bodies of work, milestones, responsibilities and budgets; uses Task/Process contracts where applicable |
| `task` | Platform candidate | Individual assigned work, goals, progress and due dates; not Temporal jobs or scheduler triggers |
| `booking` | Domain candidate | Reservation lifecycle, availability and capacity for resources; neither a clock nor execution scheduler |
| `export` | Library | Safe rich-text tree conversion to export formats; does not own authored documents |
| `ui` | Presentation | Generic theme-vocabulary components, unaware of specific capability data |
| `content-editor` | Presentation | Publisher document editing through Publisher's public contract |
| `gallery` | Presentation | Publisher gallery document types over Media images |

### Organisational and activity domain candidates

| Candidate | Potential bounded responsibility / disposition |
|---|---|
| `organisation` | Organisational archetype unless independent governance data/lifecycle justify domain ownership |
| `business` | Business-operations composition/archetype; not a duplicate Identity hierarchy |
| `company` | Company/legal-entity-specific records and lifecycle only when independent of general organisation model |
| `club` | Club policies and specialist activities; common membership remains in Identity |
| `committee` | Committee decisions, votes and minutes where independently governed; otherwise archetype |
| `department` | Organisational unit type and operational metadata, not a second group tree |
| `office` | Organisational location/resource role; not a duplicate location or asset service |
| `membership` | Paid/scheme membership rights, terms, renewals and benefits; NOT Identity's existing group membership |
| `events` | Planned participant activities and event lifecycle; not background scheduled tasks |
| `course` | Structured learning offering and outcomes |
| `enrolment` | Course/learner participation and allocation |
| `training` | Verified training participation, competencies and completion records |
| `gaming` | Game-specific state, rankings and participation; separate from generic Identity accounts |
| `workshop` | Event/Course specialisation unless distinct rules justify a separate domain |
| `blog` | Publisher document type and presentation unless separate domain state demonstrably exists |
| `job` | Ambiguous: employment role/post is possible domain; background execution job belongs to Scheduler |

### Commerce and operations domain candidates

| Candidate | Potential bounded responsibility / exclusion |
|---|---|
| `product` | Saleable products, families, bundles and variants; not stock quantities |
| `pricing` | Price rules, lists, promotions and calculations; not order records |
| `inventory` | Stock quantities, movements, reservations and stock ownership; not product definition or warehouse workflows |
| `asset` | Owned controlled assets, lifecycle, assignments and valuation references; not generic sale stock |
| `order` | Commercial orders, lines, states and fulfilment obligations; not payment settlement |
| `billing` | Billable charges and periods; not the owner of the entire ordering lifecycle |
| `invoice` | Invoice issuance, numbers, adjustments and settlement state; not payment provider execution |
| `payment-gateway` | Payment-provider initiation, status, capture and refund coordination; no accounting ledger |
| `accounting` | Ledger, journal, periods and reconciliations; not the originating sales or warehouse transactions |
| `subscription` | Recurring commercial agreements, plans, renewals and entitlements; not payment provider |
| `shop` | Retail/ecommerce composition and shop-specific operations; not replacement of Product/Order/Inventory |
| `marketplace` | Multi-seller participation, listings and marketplace-specific commercial operations |
| `warehouse` | Receiving, put-away, picking, packing and physical warehouse operations; Inventory owns stock facts |
| `logistics` | Shipment and delivery operations, dispatch, carrier coordination and tracking |

### Names held for consolidation or clarification

- `image`: covered by `media` unless a separately justified boundary emerges.
- `integration-adapters`: the general `integrations` responsibility; provider adapters such as `integrations/trustpilot` are optional integrations, not necessarily separate layers.
- `tenancy`: tenant and group ownership already belongs to Identity; tenant administration UI may be presentation.
- `document`: no generic document service should duplicate completed Publisher.
- `blog` and `workshop`: favour Publisher/Event/Course composition unless independent domain ownership is established.
- `job`: distinguish employment positions from infrastructure background jobs.
- `accounting`, `billing`, `inventory`, `invoice`, `booking` and `membership` are candidates for classification review, not new Foundation obligations on completed layers.
- `organisation-model` is a proposed archetype-owning boundary; it does not change the published Identity, Configuration or Authorisation contracts.

## Consolidations

- No shared `persistence-service` (ADR-0002). Generic `user-service` and `group-service` are Identity; sessions stay in Authentication.
- `blog-service` remains a Publisher document-type/presentation candidate; `image-service` and `media-library-ui` belong with `media`; `storage-service` is `storage`; `todo-manager` is a potential Task use case.
- `iam-integration` is unrelated to the general-purpose `integrations` capability.
- `integration-adapters` is `integrations`; `policy-map` is `legal-compliance`; `doc-search` is the `search-service` candidate.
- Infrastructure `scheduling-service` and background `job-service` are Scheduler concerns; an employment `job` would be a different domain. `workflow-service` is `workflow`; `notification-service` is `notifications`; `event-bus-service` and the former `events-queue` are `message-queue`.
- Existing repository names and contracts are kept unless an independently reviewed change justifies a migration.

## Starting a capability

A repository is created only once these are established:

1. purpose and bounded responsibility;
2. owned state, rules and lifecycle;
3. explicit exclusions;
4. provided public contracts and any schema versions required by tooling;
5. required and optional capability contracts;
6. configuration and persistence-provider requirements;
7. security, privacy, accessibility and internationalisation obligations;
8. independent contract, integration and consumer acceptance tests.

The group model is defined in [Identity and Access](identity.md) ([ADR-0003](decisions/ADR-0003-group-model-and-identity-first.md)).
