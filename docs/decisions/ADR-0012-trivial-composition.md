# ADR-0012 — Trivial Composition

**Status:** Accepted  
**Date:** 2026-10-24  
**Revised:** 2026-10-25: screens follow the amended ADR-0009; every capability has a one-page host contract  
**Accepted:** 2026-10-25, with the host contract's limit raised from 150 to 200 lines

> **In short:** composing a capability into an application must be trivial and measured. A host installs it, makes one call with what it already has (a database connection, the mail sender, the access decision) and writes no adapters of its own. Every capability fills its own ports by default, every capability asks for the same few things in the same shapes, and its tests prove a minimal host composes it within a fixed budget.

## Why

Applications should remain lightweight composition roots. Without a common contract for ports and composition, each application repeats store configuration, caller resolution, access-decision wiring and operational tasks. This makes capabilities harder to integrate and verify independently.

The cost is both code and coupling: consumers should compose a capability from its public contract without reading private implementation details.

## The decision

### 1. A composition budget

Composing a capability into a host takes:

- one package install, following current accepted `master` during development;
- one call in one server plugin, passing only what the host already owns;
- no more than **50 lines** of host code for that capability, counted as every line outside the host's own screens, content and data files that names the capability.

What the host passes is limited to: connections and secrets ([ADR-0002](ADR-0002-composition-supplied-persistence-and-capability-owned-schemas.md)), configuration values, and decisions that belong to the application, such as which document types it registers. Everything else has a default.

### 2. The same shapes everywhere

Every capability asks for the shared needs in the same structural shapes, written here and copied verbatim into each contract. There is no shared runtime package: TypeScript's structural typing makes one provider fit every capability, and repository ownership does not couple to another ([ADR-0008](ADR-0008-independent-layer-delivery.md)).

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

Every capability carries `examples/minimal-host`: a Nuxt application that composes it, and nothing else, within the budget. CI builds it, runs a smoke test against it and fails when its composition code exceeds the budget. The platform test harness proves combinations against the latest accepted development revisions. Acceptance results give the figure.

### 5. Screens

[ADR-0009](ADR-0009-headless-capability-layers.md), as amended on 2026-10-25, lets a capability ship its own screens as an optional presentation part. Using them costs the host only configuration: their paths, wording overrides, or `presentation: false`. That configuration counts against the budget; the screens themselves do not.

### 6. A one-page host contract

Every capability has `CONTRACT.md` at its root: what a host needs and nothing else, in at most **200 lines**, so that a person or an agent composes it from that page alone, without reading the capability's other documents or code. Its sections, in this order:

| Section | Holds |
|---|---|
| Install | The current development dependency and one configuration example, within the budget |
| Settings | Each environment variable or configuration value, its default, and what it does |
| Host passes | What the host must supply, in the shared shapes of decision 2, and nothing with a default |
| Client and server functions | Those screens and other capabilities call, with inputs, outputs and error codes |
| Permissions | Each permission it asks about, with its risk and effect |
| Screens | Its presentation part, if any: pages, paths, message keys, and how to switch it off |
| Headless surface | What any screen must still do (ADR-0009) |
| You will hit this | Rules a host would otherwise discover by failing: identifier formats, name patterns, what needs a second person or step-up |

The capability's longer contract stays for its own developers. When they disagree, the accepted ADRs and normative capability specification govern; the documents and implementation must be reconciled before acceptance. Implementation is evidence, not architectural authority. A handover to compose a capability names its `CONTRACT.md` and nothing else of the capability's.

## Adopting it

Each capability implements the common contract and composition requirements in its own repository. Its own plan determines implementation order; consumers adopt the accepted public surface through protected pull requests and verify integration against current development dependencies (ADR-0008).

A suite's integration repository composes its members only after their independent public contracts are established. Capability-specific migration plans, metrics and implementation decisions stay with their owners.

## What follows

| Gains | Costs |
|---|---|
| Composing a capability is quick, cheap and predictable, for people and for agents | Each capability gains a PostgreSQL store, fakes and an example host to maintain |
| One access decision and one caller resolver serve every capability | The shared shapes must be agreed before the pilot, and changing one touches every contract |
| Applications hold product code, not plumbing | The budget is a line count, a proxy that can be gamed; reviewers judge the spirit |
| Combinations proven in the harness carry straight into applications | Breaking changes require immediate consumer integration and corrections |

## Guardrails

- A capability's contract lists what a host must pass, and nothing in that list has a default the capability could supply itself.
- `examples/minimal-host` composes within the budget, or CI fails.
- No host writes a store, fake or relay for a capability that complies with this ADR; a review that finds one raises it against the capability.
- The shared shapes change only by amending this ADR.
- `CONTRACT.md` exists, keeps its sections and stays within 200 lines; the repository's documentation check fails otherwise.
