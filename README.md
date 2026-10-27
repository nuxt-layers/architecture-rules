# Nuxt Layers Platform Architecture

> **In short:** the rules every `nuxt-layers` capability and application follows: principles, cross-cutting standards and Architecture Decision Records. A capability's own detail lives in its own repository.

> **AI-driven development.** Nuxt Layers is an experimental initiative in which AI does the principal architecture, development, testing, security assessment and documentation, under human direction. The project owner decides requirements, governance, acceptance and releases. Every contribution meets the same standards, whatever its origin. See [Contributing](CONTRIBUTING.md).

## Principles

The ecosystem is bounded capabilities, built as Nuxt Layers, that evolve independently and are combined into light Nuxt applications. Every capability and application:

- is organised around bounded capabilities with clear public contracts, keeping private implementation inside each layer;
- keeps dependencies one-way and free of cycles, and combines capabilities rather than building inheritance chains;
- keeps infrastructure replaceable where practical;
- enforces security on the server, to **OWASP ASVS 5.0** Level 2 at least, with Level 3 for security-sensitive operations;
- designs for privacy from the start;
- meets **WCAG 2.2** Level AA, with semantic markup, and lets people customise presentation without breaking accessibility;
- designs for internationalisation from the start, with **BCP 47** language tags and **Unicode CLDR** locale data through **ECMA-402** (the `Intl` API).

## Which document wins

Where documents conflict, authority descends in this order:

1. an accepted Architecture Decision Record, for the decision it settles;
2. the [Architecture](docs/architecture.md) overview;
3. the [Layer Standard](docs/layer-standard.md);
4. the cross-cutting baselines: identity, security, privacy, presentation;
5. a capability's own normative documents;
6. an application's documents;
7. the implementation, which is evidence of behaviour, not authority.

## Where things live

This repository holds only what applies to the whole ecosystem. A capability's contracts, data model, threat model, plan and processes live in its own repository; a suite's architecture and cross-capability processes live in its integration repository, such as `nuxt-layers/iam-integration` ([ADR-0004](docs/decisions/ADR-0004-documentation-placement.md)). Documents elsewhere are listed in [Resources](RESOURCES.md). This repository holds no plans or status ([ADR-0010](docs/decisions/ADR-0010-documentation-style.md)).

## Documents

| Document | Covers |
|---|---|
| [Architecture](docs/architecture.md) | The model, invariants, deployment, routes, security and privacy in brief |
| [Layer Standard](docs/layer-standard.md) | What a layer is, its repository, contracts, manifest, tests, consumption and versioning |
| [Identity and Access](docs/identity.md) | Identity, groups, memberships, ownership and authorisation |
| [Security](docs/security.md) | The security baseline and data-store controls |
| [Privacy](docs/privacy.md) | Data protection by design |
| [Presentation, Internationalisation and Accessibility](docs/presentation.md) | Design tokens, languages, WCAG and the screen checklist |
| [Capability Catalogue](docs/catalogue.md) | The capabilities that exist, the candidates, and the rules for starting one |
| [Decisions](docs/decisions/) | One file per Architecture Decision Record, in number order |
| [Resources](RESOURCES.md) | Every external document relied on, pinned |
