# ADR-0008 — Independent Layer Delivery

**Status:** Accepted  
**Date:** 2026-10-11  
**Amended:** 2026-10-26: brought in line with ADR-0010 and later decisions
**Accepted:** 2026-10-23

> **In short:** each layer is finished in its own repository against versioned contracts, with one draft pull request per repository, and the suite is composed and verified at release rather than on every change.

## Why

[ADR-0001](ADR-0001-capability-repositories-and-composition-shells.md) gives each capability its own repository, and [ADR-0005](ADR-0005-iam-suite.md) connects the IAM suite's members only through ports and an integration repository. In practice, building the suite drifted towards changing every member at once: each phase opened pull requests in four or five repositories, recomposed `platform-test-harness` against each merge commit, and watched every pull request until CI was green. That cost far more AI tokens and CI minutes than the work needed, and it tied the members' release cadence together.

A monorepo was considered and rejected: it would make the members one large layer in practice, and lose the independence ADR-0001 exists to protect.

## The decision

Each layer is finished in its own repository against versioned contracts. The suite is composed and verified at release, not on every change.

1. **One layer at a time.** A change is delivered in the repository that owns it, against the other members' published contracts. Members never import one another; they connect only through the integration repository's adapters. A change that needs another member's contract to change waits for that member's release.
2. **Versioned ports.** From 1.0, every capability follows semantic versioning. A change that breaks its contract needs a new contract version and a major release; consumers and adapters pin a tag, never a moving branch. Each release is listed in the repository's `CHANGELOG.md`.
3. **Composition at release.** `platform-test-harness` is pinned to released tags, and is recomposed and run when a member releases, not for each pull request. A member's own CI proves its contract with its own tests and conformance suites.
4. **Design before code.** Design questions are put to the project owner before code is generated ([Contributing](../../CONTRIBUTING.md)).
5. **One draft pull request per repository** for each piece of work. Its CI is fixed until green.
6. **Minimal monitoring.** Pull requests are not watched or polled by default. The project owner reports merges and asks for follow-up when it is needed.
7. **Reproducible builds.** CI uses the ECR Public mirror of PostgreSQL (`public.ecr.aws/docker/library/postgres`). Lockfiles record Git dependencies as `codeload.github.com` tarballs, never `git@github.com`.

## What follows

| Gains | Costs |
|---|---|
| Each piece of work touches the fewest repositories, which reduces tokens, CI minutes and review effort | A defect that only shows when members are composed is found at release rather than at merge. Conformance suites and adapter type checks narrow the gap |
| Members release on their own cadence; a host upgrades one member at a time | A cross-member change takes one release per member, in dependency order |
| Contracts become the real boundary, because nothing else can be relied on between releases | Unwatched pull requests depend on the project owner to report failures and merges |

## Guardrails

- The integration repository's adapters state structurally what they need of each member, so a host's type check shows when a release no longer fits.
- A member's release that changes a port's shape is a major release, and the harness is recomposed against it before the integration repository's pin moves.
- [ADR-0004](ADR-0004-documentation-placement.md) and [ADR-0010](ADR-0010-documentation-style.md) still apply: other repositories' documents are named through `RESOURCES.md`, pinned to a tag or commit.
