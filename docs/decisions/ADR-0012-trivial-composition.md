# ADR-0012 — Trivial Composition

**Status:** Accepted  
**Date:** 2026-10-24  
**Revised:** 2026-10-25: screens follow the amended ADR-0009; every capability has a one-page host contract  
**Accepted:** 2026-10-25, with the host contract's limit raised from 150 to 200 lines

> **In short:** composing a capability into an application must be trivial and measured. A host installs it, makes one call with what it already has (a database connection, the mail sender, the access decision) and writes no adapters of its own. Every capability fills its own ports by default, every capability asks for the same few things in the same shapes, and its tests prove a minimal host composes it within a fixed budget.

## Why

[ADR-0001](ADR-0001-capability-repositories-and-composition-shells.md) makes applications *lightweight composition roots*. In practice they are not. Measured on 24 October 2026 with a coupling map of the released layers:

| Finding | Evidence |
|---|---|
| The capabilities are well separated | No capability imports another, apart from `publisher` using `rich-text`. Each declares its own ports |
| The hosts are not light | Westray-Papa-Westray holds about 1,850 lines in 23 files that touch a capability; the platform test harness about 1,050 in 14 |
| The same work is repeated | Each host writes its own theme store, its own access policies, its own mapping from the signed-in person to each capability's idea of a caller, its own outbox relays and maintenance timers |
| Pairs need their own glue | Each capability asks for "who is calling" and "may they" in its own shape, so Authorisation must be adapted to each capability separately; `iam-integration` does this once for the IAM suite, yet each host still writes over 300 lines around it |

The cost is not only code. Every composition sends a person or an AI agent through several repositories' internals, and that is what makes integration slow and expensive.

## The decision

### 1. A composition budget

Composing a capability into a host takes:

- one package install, pinned to a release;
- one call in one server plugin, passing only what the host already owns;
- no more than **50 lines** of host code for that capability, counted as every line outside the host's own screens, content and data files that names the capability.

What the host passes is limited to: connections and secrets ([ADR-0002](ADR-0002-composition-supplied-persistence-and-capability-owned-schemas.md)), configuration values, and decisions that belong to the application, such as which document types it registers. Everything else has a default.

### 2. The same shapes everywhere

Every capability asks for the shared needs in the same structural shapes, written here and copied verbatim into each contract. There is no shared runtime package: TypeScript's structural typing makes one provider fit every capability, and no release couples to another ([ADR-0008](ADR-0008-independent-layer-delivery.md)).

| Need | Shape |
|---|---|
| Database | A `pg`-compatible pool, from which the capability opens its own connections and owns its schema (ADR-0002) |
| Caller | `resolveCaller(event) → { tenantId, subjectId \| null, assurance }` |
| Access | `decide(caller, action, resource) → { allowed, reason }`, failing closed |
| Mail | `send({ to, template, data })` |
| Events | The capability's outbox, drained by `relay()`; the host never reads its tables |
| Scheduled work | `tasks: { name, every, run }[]`, run by the host's one scheduler |
| Clock | `now()`, defaulting to the system clock |

### 3. Capabilities fill their own ports

Each capability ships, in its own package:

- a PostgreSQL implementation of its stores, used when it is given a pool;
- fakes of every port for tests, refused outside test configuration (ADR-0002, decision 2);
- a recording mail sender and caller resolver for tests and demonstrations.

Where capabilities belong together, one package wires them once and returns the shared shapes. `iam-integration` becomes that package for the IAM suite: given a pool and a mail sender, it returns `resolveCaller`, `decide` and `tasks`, ready for any capability, IAM or not.

### 4. Proven, not promised

Every capability carries `examples/minimal-host`: a Nuxt application that composes it, and nothing else, within the budget. CI builds it, runs a smoke test against it and fails when its composition code exceeds the budget. The platform test harness proves combinations using only these published compositions. Release notes give the figure.

### 5. Screens

[ADR-0009](ADR-0009-headless-capability-layers.md), as amended on 2026-10-25, lets a capability ship its own screens as an optional presentation part. Using them costs the host only configuration: their paths, wording overrides, or `presentation: false`. That configuration counts against the budget; the screens themselves do not.

### 6. A one-page host contract

Every capability has `CONTRACT.md` at its root: what a host needs and nothing else, in at most **200 lines**, so that a person or an agent composes it from that page alone, without reading the capability's other documents or code. Its sections, in this order:

| Section | Holds |
|---|---|
| Install | The pinned dependency and one configuration example, within the budget |
| Settings | Each environment variable or configuration value, its default, and what it does |
| Host passes | What the host must supply, in the shared shapes of decision 2, and nothing with a default |
| Client and server functions | Those screens and other capabilities call, with inputs, outputs and error codes |
| Permissions | Each permission it asks about, with its risk and effect |
| Screens | Its presentation part, if any: pages, paths, message keys, and how to switch it off |
| Headless surface | What any screen must still do (ADR-0009) |
| You will hit this | Rules a host would otherwise discover by failing: identifier formats, name patterns, what needs a second person or step-up |

The capability's longer contract stays for its own developers. When the two disagree, the code and the longer contract are right and `CONTRACT.md` is a bug. A handover to compose a capability names its `CONTRACT.md` and nothing else of the capability's.

## Adopting it

| Order | Capability | Because |
|---|---|---|
| 1 | `theme-manager` | Small, and its hosts each write a store and a policy (about 590 lines in Westray-Papa-Westray) |
| 2 | `publisher` | The most host code in Westray-Papa-Westray (about 790 lines); `content-editor` is built to this rule from the start |
| 3 | `iam-integration` with the IAM suite | Removes the per-host caller mapping and access policies for every other capability |
| 4 | `configuration` | Its administration depends on the IAM adapters |

Adding defaults and the minimal host is a minor release. Removing a provider function a host used is a major one (ADR-0008). Applications adopt each release as it lands, and each block's PR reports the drop in host code.

## What follows

| Gains | Costs |
|---|---|
| Composing a capability is quick, cheap and predictable, for people and for agents | Each capability gains a PostgreSQL store, fakes and an example host to maintain |
| One access decision and one caller resolver serve every capability | The shared shapes must be agreed before the pilot, and changing one touches every contract |
| Applications hold product code, not plumbing | The budget is a line count, a proxy that can be gamed; reviewers judge the spirit |
| Combinations proven in the harness carry straight into applications | Released capabilities need a minor, and some a major, release to comply |

## Guardrails

- A capability's contract lists what a host must pass, and nothing in that list has a default the capability could supply itself.
- `examples/minimal-host` composes within the budget, or CI fails.
- No host writes a store, fake or relay for a capability that complies with this ADR; a review that finds one raises it against the capability.
- The shared shapes change only by amending this ADR.
- `CONTRACT.md` exists, keeps its sections and stays within 200 lines; the repository's documentation check fails otherwise.
