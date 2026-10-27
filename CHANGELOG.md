# Changelog

Major changes, newest first. This repository has no releases; each entry is dated.

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
- One architecture overview (`docs/architecture/architecture.md`) replaces the platform architecture and the composition and dependency model.
- One layer standard (`docs/standards/layer-standard.md`) replaces the layer architecture, layer interface standard, capability manifest, layer repository standard, layer consumption workflow and compatibility and versioning standard.
- Documents consolidated: `docs/RESOURCES.md`, `docs/PLAN.md`, `CONTRIBUTING.md` (replacing `AI_DEVELOPMENT.md`) and this changelog added; the headless migration plan removed; the capability catalogue reduced to existing capabilities.
- ADR-0010 amended: documents that stay true without upkeep.
- ADR-0012 accepted, with `CONTRACT.md` limited to 200 lines.
- ADR-0009 amended: a headless core with an optional presentation part.
