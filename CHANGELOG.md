# Changelog

Major changes, newest first. This repository has no releases; each entry is dated.

## 2026-11-08

- Layer Standard §5: where each capability's `CONTRACT.md` is found, and that a new layer requires an owning capability from the catalogue instead of reimplementing it, reading only the contracts its manifest requires.

## 2026-11-07

- Catalogue: `events-queue` is renamed `message-queue`, so it is not confused with calendar events such as meetings and workshops (the `event-service` candidate).

## 2026-11-06

- Catalogue: the proposed capabilities now have repositories and are listed as capabilities, with a brief purpose each. `policy-map` is renamed `legal-compliance` (legal duties by jurisdiction, such as UK GDPR and the Online Safety Act); `doc-search` returns to the `search-service` candidate; `integration-adapters` is `integrations`.

## 2026-11-05 (site hosting)

- Catalogue: `site-hosting` proposed, for custom domains and finding a request's site.

## 2026-11-05

- Catalogue: candidates proposed in Publisher's design review (scheduler, workflow, content security, export, privacy, policy map, document search, translation, metadata, notifications, events queue, comments, integrations, and a suggested SEO layer), with older candidate names folded into them.

## 2026-11-04

- Hosting and storage providers may be outside the UK where UK data-protection law permits the transfer, with each provider's security verified independently and recorded by each application (ADR-0014); the privacy baseline states that UK law governs; ADR-0010 amended for providers in an application's `RESOURCES.md`.

## 2026-10-31

- Reconciled remaining documentation contradictions: release-era instructions, inaccessible historical references, contract authority, repository file placement, manifest metadata and continuous integration acceptance.


- Removed inaccessible historical document links from `RESOURCES.md` and aligned references and documentation rules with the current accessible repositories.


- Adopted continuous ecosystem development: latest accepted `master`, affected-consumer acceptance testing and fix-forward integration; suspended formal release and dependency-tag requirements pending a future release decision (ADR-0008 and dependent standards).
- Aligned the Layer Standard, composition, documentation and contributing guidance with the development policy.
- Established Node 26 LTS and pnpm 12 as the development and CI baseline (ADR-0013), distinct from consumer minimum runtime requirements.

## 2026-10-26

- The organisation renamed from `nuxt4-layers` to `nuxt-layers`, and the package scope from `@nuxt4-layers/` to `@nuxt-layers/`, so neither names a Nuxt release.
- Slash-joined phrases in the security baseline written out in full.
- Final readability pass: the README rewritten around principles, authority and a table of documents; *In short* lines added where missing; the privacy and presentation baselines tidied; the Markdown check now requires an *In short* in every document under `docs/`.
- `docs/decisions/README.md` removed: the README points to the folder of decisions instead of a hand-kept index.
- The documentation style standard folded into ADR-0010.
- `PLAN.md`, where a repository has one, also lives at the root; ADR-0010's table of files that stay true tidied.
- `RESOURCES.md` moved to the repository root, beside `README.md`, `CHANGELOG.md` and `CONTRACT.md` (ADR-0010 amended).
- ADRs reviewed for consistency: ADR-0001 to ADR-0008 follow ADR-0010's order (*In short*, Why, The decision, What follows); references to replaced documents updated; ADR-0004 and ADR-0010 amended for `docs/RESOURCES.md` and `CONTRACT.md`; ADR-0011 amended for the amended ADR-0009 and content-editor's design.

## 2026-10-25

- Internationalisation standards named (BCP 47, Unicode CLDR through ECMA-402, W3C Internationalization); the README's principles name OWASP ASVS and the internationalisation standards alongside WCAG.
- `docs/PLAN.md` removed: this repository holds no plans or status (ADR-0010 amended); candidate capabilities moved to the catalogue.
- Documents moved to `docs/` without version suffixes or single-file folders; the decision index moved to `docs/decisions/README.md`.
- `docs/identity/identity.md` replaces the identity and authorisation architecture and the group model definition; `docs/security/security.md` replaces the security architecture and absorbs the data store security standard.
- One architecture overview (`docs/architecture/architecture.md`) replaces the architecture rules and the composition and dependency model.
- One layer standard (`docs/standards/layer-standard.md`) replaces the layer architecture, layer interface standard, capability manifest, layer repository standard, layer consumption workflow and compatibility and versioning standard.
- Documents consolidated: `docs/RESOURCES.md`, `docs/PLAN.md`, `CONTRIBUTING.md` (replacing `AI_DEVELOPMENT.md`) and this changelog added; the headless migration plan removed; the capability catalogue reduced to existing capabilities.
- ADR-0010 amended: documents that stay true without upkeep.
- ADR-0012 accepted, with `CONTRACT.md` limited to 200 lines.
- ADR-0009 amended: a headless core with an optional presentation part.
