# ADR-0008 — Continuous Development and Layer Integration

**Status:** Accepted  
**Date:** 2026-10-11  
**Amended:** 2026-10-31: operational experience supersedes release-based integration during active development

> **In short:** until formal releases begin, each capability's latest accepted `master` is authoritative. Every change is integrated and tested across affected consumers, and incompatibilities are fixed forward.

## Why

Independent release tags and delayed composition have imposed unnecessary administration during active development and deferred the discovery of cross-layer breakage. The ecosystem needs independent repository ownership without postponing integration.

## The decision

During pre-release development:

1. **Current development source.** Every layer and application consumes the latest accepted `master` of its dependencies. Superseded development revisions are not compatibility targets.
2. **Immediate propagation.** A provider change merged into `master` triggers refresh and acceptance testing in affected consuming layers, integration repositories and applications. Adoption begins immediately; it is complete only when the affected consumers have resolved their dependencies to the current provider `master` and passed their required checks. Failures remain visible until corrected.
3. **Fix forward.** Breaking changes found in integration are repaired in affected consumers or, where the provider is defective, in the provider. Do not hold consumers on older development revisions merely to avoid integration work.
4. **No formal development releases.** New release packages, release tags and release-based dependency pins are suspended until the project owner explicitly approves a release lifecycle. Existing tags and releases remain historical records.
5. **Independent ownership.** Changes remain focused in the repository that owns them, delivered through protected-branch pull requests. Public contracts, ports and capability boundaries remain mandatory.
6. **Reproducible execution, not frozen development.** Dependency tooling may record commit resolutions internally and CI may use frozen lockfiles. These records must be refreshed as part of propagation and must not be treated as reasons to defer adopting the latest `master`.
7. **Cost discipline.** Affected consumers, rather than every unrelated repository, are tested. Pull requests are not continuously polled or monitored by AI agents by default; merge-triggered automation may perform the required refresh and checks.
8. **Security and design.** Design decisions remain subject to human acceptance, and security, quality and contract verification remain required throughout development.

When formal releases are approved, a separate accepted decision will define versioning, release compatibility, publishing and migration requirements.

## Adopting it

Update dependency declarations, workflows, contracts and consumer harnesses to follow and test current `master`. Automate propagation where practical, with failures reported for correction rather than silently tolerated.

## What follows

| Gains | Costs |
|---|---|
| Breaking integration changes surface promptly | A provider merge may require follow-up work in several consumers |
| No release administration during active development | Integration automation and ownership must be maintained |
| Latest accepted code is used throughout the ecosystem | Consumers can temporarily fail until fixes are merged |
| Capability boundaries and repository independence remain | Acceptance testing must establish the integrated state |

## Guardrails

- A passing provider CI run does not itself prove consumer compatibility. Acceptance reports identify each affected consumer, its check result and any outstanding integration failure.
- No automatic forced pushes, unreviewed merges or weakening of protected branches.
- Lockfile or tool-generated commit identifiers do not constitute a formal pinning policy.
- Historical architectural references in `RESOURCES.md` remain revision-specific under ADR-0004 and ADR-0010.
