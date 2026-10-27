# ADR-0004 — Documentation Placement

**Status:** Accepted  
**Date:** 2026-10-09  
**Amended:** 2026-10-23: ADRs follow ADR-0010; no glossary; the group model stays here  
**Amended:** 2026-10-26: external links and pins only in `RESOURCES.md`, with no pointer files ([ADR-0010](ADR-0010-documentation-style.md))

> **In short:** a document lives with whoever keeps it true: ecosystem-wide rules and decisions here, a capability's detail in its own repository, a suite's cross-capability processes in its integration repository.

## Why

This repository holds the ecosystem's normative architecture. As capabilities are designed, there is pressure to place their detailed specifications here too: data models, contracts, process steps and threat models. Doing so would mix ecosystem-wide rules with capability-specific detail, duplicate what capability repositories already document, and let the two drift apart.

Capability repositories already maintain their own contract, threat model and plan ([Layer Standard](../layer-standard.md) §3). Several capabilities also form suites whose processes span more than one of them, such as the Identity and Access Management (IAM) suite.

## The decision

A document lives with whoever is responsible for keeping it true.

| Location | Holds |
|---|---|
| `nuxt-layers/platform-architecture` | Ecosystem-wide principles and invariants; cross-cutting standards; Architecture Decision Records; the capability catalogue as an index of one line per capability |
| Each capability repository, under `docs/` | Its `CONTRACT.md` and longer contract, data model, threat model and control register, `PLAN.md`, and the processes it owns |
| A suite's integration repository, e.g. `nuxt-layers/iam-integration`, under `docs/` | The suite's architecture and the processes that span several of its capabilities |

Rules:

1. **ADRs stay short.** An ADR follows [ADR-0010](ADR-0010-documentation-style.md)'s order and links to detail. It names the detailed document that carries the decision out; that document is authoritative because the ADR names it.
2. **Another repository's documents are named through `RESOURCES.md`**, pinned to a tag or commit, so a decision refers to exactly the version that was reviewed. A material change to the detail requires a new or amended ADR that moves the pin.
3. **Each capability's `RESOURCES.md` lists the ADRs that govern it**, so references run in both directions.
4. **Ecosystem-wide rules are not restated** in capability repositories; they link here.
5. **Detail that moves or is removed leaves no pointer file.** Its last version is listed in `RESOURCES.md` at a pinned commit, for the record.

The documentation authority order in the README is unchanged: an ADR outranks the detailed document it names.

## What follows

| Gains | Costs |
|---|---|
| This repository stays small enough to read in full | Readers follow links across repositories, some of them private |
| Capability detail is maintained beside the code that implements it | Changing pinned detail requires an ADR update |
| Pinned links make every decision reproducible |  |

## Guardrails

- The Markdown check rejects external links outside `RESOURCES.md`, and links there to a `nuxt-layers` repository that point at a moving branch.
- The [group model](../identity.md) stays here: it binds Identity, Authorisation and every capability that uses groups ([ADR-0003](ADR-0003-group-model-and-identity-first.md)).
- The capability catalogue is an index; definitions longer than one line belong in the capability's own README.
