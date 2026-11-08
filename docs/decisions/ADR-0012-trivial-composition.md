# ADR-0012 — Trivial Composition

**Status:** Accepted  
**Date:** 2026-10-24  
**Revised:** 2026-10-25: screens follow the amended ADR-0009; every capability has a one-page host contract  
**Accepted:** 2026-10-25, with the host contract's limit raised from 150 to 200 lines  
**Amended:** 2026-10-31: replace the line ceiling with a concise, complete, independently usable consumer contract  
**Amended:** 2026-11-05: files join the shared shapes, filled by `storage`  
**Proposed amendment:** 2026-11-08: define shared caller, decision and scheduled-task semantics (subject to acceptance)

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

What the host passes is limited to: services in the shared shapes of decision 2, connections and secrets ([ADR-0002](ADR-0002-composition-supplied-persistence-and-capability-owned-schemas.md)), configuration values, and decisions that belong to the application, such as which document types it registers. Everything else has a default.

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
| Files | The `StorageService` of `storage`'s contract: the host passes `storageFor('<consumer>')`; the consumer keeps each item's record and storage only the bytes. The consumer never imports `storage` |

### 2a. Shared caller, access and scheduled-task semantics

A capability declares only the structural services it needs. It never imports a particular caller provider, decision engine, integration suite or scheduler. The composition root connects implementations. The following shapes define common interoperation; each capability may retain additional domain-specific services only when their purpose cannot be expressed by these shapes.

**Caller resolution.** `resolveCaller(event)` returns a trustworthy, server-derived context. It must never accept a client-asserted subject, assurance or tenant as proof. Resolution errors reject. An unauthenticated request has `subjectId: null` and `assurance: null`, never a fabricated identity.

```ts
type Assurance = {
  level: 'aal1' | 'aal2'
  phishingResistant: boolean
  authenticatedAt: string | null // UTC ISO 8601 instant; null for unauthenticated calls
}
type Caller =
  | { tenantId: string | null; subjectId: null; assurance: null }
  | { tenantId: string | null; subjectId: string; assurance: Assurance }
type ResolveCaller<Event = unknown> = (event: Event) => Promise<Caller>
```

`tenantId` is a trusted tenant context, **not** a claim that the subject is authorised in that tenant. It may be null where no trusted tenant is resolved; tenant-scoped protected operations refuse such a caller. Providers must not invent a tenant or reuse one from an unrelated request. `authenticatedAt` records the most recent qualifying authentication or step-up, not the session's initial creation time. When missing, expired, invalid or untrustworthy, freshness-dependent operations deny. Assurance can never be elevated by a downstream adapter. No default caller resolver signs anybody in.

**Access decisions.** Every protected operation passes the resolved caller, the action and a resource with an unambiguous owner/tenant scope. The decision engine, not the requesting capability, determines permissions, membership effects and assurance sufficiency. A missing decision provider, exception or unknown result denies the operation; a capability must never convert ambiguity into an allow.

```ts
type Resource = {
  type: string
  id: string
  tenantId: string | null
  owningGroupId?: string | null
}
type StepUpRequirement = {
  minimumLevel: 'aal1' | 'aal2'
  phishingResistant: boolean
  maxAuthenticationAgeSeconds: number | null
}
type Decision =
  | { allowed: true }
  | { allowed: false; reason: 'not-permitted' }
  | { allowed: false; reason: 'insufficient-assurance'; requirement: StepUpRequirement }
type Decide = (caller: Caller, action: string, resource: Resource) => Promise<Decision>
```

The decision provider verifies the resource's **authoritative** scope; a caller-supplied `resource.tenantId` or `owningGroupId` is not sufficient evidence of tenancy or ownership. All decisions fail closed across tenant boundaries. Explicitly denying an unknown resource must not expose whether that resource exists. `insufficient-assurance` is only used where its disclosed requirement is permitted; other denial reasons are reduced to `not-permitted` at capability boundaries that must not reveal policy detail. A decision is not a substitute for the domain's transaction, invariant checks or independent governance-approval policy.

**Scheduled work.** A capability describes tasks independently of their executor. The currently accepted minimal shape remains `{ name, every, run }[]`; `every` is a five-field cron expression and `run` is an asynchronous, server-only handler. Names are stable and namespaced by the declaring capability. Registration of duplicate names with conflicting definitions is refused. Neither registering nor importing a capability starts its tasks.

```ts
type ScheduledTask = {
  name: string
  every: string // standard five-field cron expression, interpreted in UTC
  run(): Promise<unknown>
}
```

The executing scheduler owns registration, dispatch, coordination, retries, operational visibility and any execution metadata. It never owns or imports task-specific business logic or reads a capability's private tables. The composition root registers descriptors using the shared shape. The scheduler must not imply exactly-once execution: tasks must tolerate retries and duplicate invocations, and must arrange idempotency or concurrency protection appropriate to their domain. A failed outbox delivery must remain pending, rather than being acknowledged by the scheduler.

This minimal descriptor does **not** standardise retry counts, timeouts, cancellation signals, per-task concurrency, distributed leases, time zones beyond UTC or an interval/one-shot syntax. Those are Scheduler-owned execution design decisions or future explicit amendments, not implicit fields to be invented by individual capabilities. Calendar/date calculations must follow the platform's existing temporal semantics rather than duplicating a date/time capability.

The common shapes here are a cross-capability baseline. A suite's integration package may adapt its members to them but owns no member's permission policy, identity records, database or scheduled business work. Existing more specific public interfaces may be retained during migration; adoption must preserve their assurance checks, independent approvals, event reliability and isolation. A changed shared shape requires affected consumer integration and security acceptance under ADR-0008.

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

### 6. A self-contained consumer contract

Every capability has a root `CONTRACT.md` containing the **shortest complete description of its public integration surface**. A consuming person or agent MUST be able to install, configure, compose and use the capability from that document alone, without reading other documentation or private source. There is no fixed line limit: omit repetition, internal design and implementation history, but never omit information required for correct use. Its sections, in this order:

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

Detailed internal specifications stay under `docs/` for maintainers and do not define a competing consumer interface. The public contract contains precise available imports, required port shapes, callable signatures, inputs, outputs, failure codes and relevant operational requirements; examples must not conceal required configuration or undefined dependencies. If the public contract, internal specification or implementation disagree, reconcile them against the accepted ADRs and normative capability requirements before acceptance. Implementation is evidence, not architectural authority. A handover to compose a capability names its `CONTRACT.md` and nothing else of the capability's.

## Adopting it

Each capability implements the common contract and composition requirements in its own repository. Its own plan determines implementation order; consumers adopt the accepted public surface through protected pull requests and verify integration against current development dependencies (ADR-0008).

A suite's integration repository composes its members only after their independent public contracts are established. Capability-specific migration plans, metrics and implementation decisions stay with their owners.

## What follows

| Gains | Costs |
|---|---|
| Composing a capability is quick, cheap and predictable, for people and for agents | Each capability gains a PostgreSQL store, fakes and an example host to maintain |
| One access decision and one caller resolver serve every capability | The shared shapes must be agreed before the pilot, and changing one touches every contract |
| Applications hold product code, not plumbing | Consumer contracts require review for both completeness and unnecessary detail |
| Combinations proven in the harness carry straight into applications | Breaking changes require immediate consumer integration and corrections |

## Guardrails

- A capability's contract lists what a host must pass, and nothing in that list has a default the capability could supply itself.
- `examples/minimal-host` composes within the budget, or CI fails.
- No host writes a store, fake or relay for a capability that complies with this ADR; a review that finds one raises it against the capability.
- The shared shapes change only by amending this ADR.
- `CONTRACT.md` exists and keeps its required sections. Documentation checks validate structure and reference integrity, not an arbitrary line count.
- A minimal consumer integration or contract acceptance test MUST demonstrate that the contract is sufficient without reading the capability's other documents or private source; missing necessary information fails acceptance.
- Public API changes update `CONTRACT.md` in the same change. Consumers receive that document alone as their documentation handover.
