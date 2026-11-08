# ADR-0013 — Development Toolchain Baseline

**Status:** Accepted  
**Date:** 2026-10-31  
**Amended:** 2026-11-08: included TypeScript 6.0.3 in the development baseline

> **In short:** active development and CI target Node.js 26 LTS, pnpm 12 and TypeScript 6.0.3, with per-repository compatibility checks; consumer runtime requirements are verified separately.

## Why

A common, supported toolchain simplifies ecosystem integration and reduces exposure to outdated development tooling. A capability's development environment is not necessarily the minimum runtime it must impose on a host application.

## The decision

1. The ecosystem's preferred development and CI baseline is Node.js 26 LTS, pnpm 12 and TypeScript 6.0.3. Adopt further supported patch releases after compatibility verification.
2. Capability repositories declare their package-manager and TypeScript development requirements and use the accepted baselines locally and in CI after compatibility verification. TypeScript 6.0.3 is a development target, not a consumer runtime requirement.
3. Consumer minimum Node.js requirements follow the capability's actual runtime dependencies and Nuxt's documented requirements; do not raise them solely to match the development baseline.
4. Migrations proceed by affected repository, with contract, integration, security and acceptance checks. Where the accepted TypeScript baseline is not yet compatible with a repository's toolchain, record the incompatibility and retain its functioning development configuration until the dependency is resolved. A green toolchain upgrade in one repository does not certify all consumers.
5. CI must honour organisation Actions restrictions; third-party actions are not introduced merely for package-manager setup. Full SHA pins remain required for permitted actions.

## Adopting it

Pilot the baseline in `authorisation`, then roll it out across active capability repositories, integration repositories and consuming applications. Record implementation details in those repositories, not here.

## What follows

| Gains | Costs |
|---|---|
| Supported and consistent development tooling | Dependency compatibility needs verification |
| Fewer obsolete runtime and package-manager configurations | Migration touches multiple repositories |

## Guardrails

- Do not weaken security checks to complete a tooling migration.
- Maintain consumer compatibility only where it is explicitly declared and tested.
