# ADR-0016 — Controlled Layer Specialisation and Composition

**Status:** Accepted  
**Date:** 2026-11-12

> **In short:** explicit contract dependencies remain the default. Declarative specialisation and deliberate Nuxt Layer `extends` are permitted when they preserve independent capability ownership, public boundaries, security and an acyclic dependency graph. This decision changes no existing capability contract.

## Why

Accepted ADR-0015 separately establishes framework-independent infrastructure integration libraries, including Temporal Gateway, as distinct from Nuxt capability layers. ADR-0016 concerns controlled Nuxt inheritance and declarative specialisation and does not change ADR-0015's package-classification, contract, testing or ownership rules.

ADR-0001 and ADR-0012 already permit one-way, contract-based dependencies connected by the application composition root. The Architecture overview nevertheless prohibits any chain of Nuxt Layers extending other layers. That blanket restriction unnecessarily prevents deliberate reuse of presentation, configuration and application compositions, especially for specialised organisational archetypes.

Nuxt `extends` merges framework contributions; it is **not** domain inheritance, delegation of an authoritative service, or a substitute for a documented public contract. Inheriting code or configuration does not grant rights over its owner's state or business rules.

## The decision

1. **Contract consumption is the default.** A capability MAY require or optionally consume another capability's documented public service, port, event or data contract. The composition root supplies or resolves the provider as required by ADR-0012. Existing dependency declarations, contract-only integration and independently owned persistence remain mandatory.
2. **Declarative specialisation is permitted.** Archetypes or configuration can specialise an organisational type, workflow, presentation or application composition through explicit, validated extension points. An archetype does not own Identity groups, grant permissions or substitute for an applicable domain capability.
3. **Controlled Nuxt inheritance is permitted.** A presentation layer, application composition, or another layer with a justified public extension surface MAY use Nuxt `extends` to reuse documented framework contributions. The extending layer MUST document which exported contributions it inherits and how precedence, overrides, route ownership, automatic registration and disabling behave. A Nuxt extension does not automatically make private files, database schemas, persistence adapters or undocumented configuration public.
4. **No ownership transfer.** An extending or specialising consumer MUST NOT override an authoritative capability's security checks, data invariants, persistence ownership, events/outbox ownership or access decisions. Headless cores remain independent of presentation (ADR-0009). Existing contracts, including Identity, Authorisation, Profile, Publisher and Theme Manager, are unchanged.
5. **Explicit graph and compatibility.** All required and optional contract dependencies and direct Nuxt extension edges MUST be recorded with direction in the relevant contract and manifest metadata (using an explicit documented extension field where the manifest schema supports it; otherwise a documented supplementary declaration until a manifest schema amendment is accepted). Treat all edges together when checking cycles; extensions must never create indirect cycles. Providers do not depend upon their consumers.
6. **Composition budget is preserved.** For each capability the ADR-0012 host budget and one-call composition obligation still apply. An extension cannot conceal extra host adapters, stores, undocumented setup or registrations. Count all host code that names the capability or is required solely to compose it. A reused presentation/application shell may expose one public composition entry point but may not silently install or initialise a capability whose prerequisites are unsatisfied.
7. **Testing and change propagation.** An extending consumer MUST prove the documented inherited behaviour, precedence, disabled-presentation mode where applicable, absence of duplicate routes/plugins, security boundaries and minimal-host composition. Integration follows the latest accepted `master` and fix-forward rules in ADR-0008. Optional edges MUST have a documented absence mode; required dependencies fail closed when unavailable.
8. **Preserve the authority hierarchy.** A consumer cannot gain an undocumented right through Nuxt's merge semantics. Conflicting inherited definitions require explicit resolution in the consuming composition, with source capability contracts and accepted ADRs winning over convenient framework defaults.

## Mechanisms and intended uses

| Relationship | Typical use | Allowed dependency |
|---|---|---|
| Contract dependency | Warehouse consumes Inventory stock operations | Required or optional public contract, normally wired by host |
| Declarative specialisation | Greengrocer archetype specialises retail configuration | Schema-validated descriptions and published extension points |
| Nuxt `extends` | Retail application specialises a reusable presentation/composition shell | Deliberate framework inheritance with documented inherited surface |
| Integration adapter | Trustpilot connects to Reviews | Provider-neutral public interface; optional external provider |

Prefer composition and explicit public contracts to long inheritance chains. Nuxt `extends` is not the default mechanism for domain-to-domain integration.

## Adopting it

This ADR is **permissive**, not a migration instruction. Completed layers and their `CONTRACT.md`, manifests and implementation MUST NOT be changed solely because controlled inheritance is now allowed. Any later change to a completed capability requires its own independently justified review.

The general rules in ADR-0001, ADR-0002, ADR-0008, ADR-0009, ADR-0012 and ADR-0015 remain in force except the former categorical ban on Nuxt extension in Architecture §2 invariant 5, which this decision expressly replaces. It does not remove host composition, public-contract encapsulation, the modular-monolith default, or the acyclic graph requirement.

## What follows

| Gains | Costs |
|---|---|
| Deliberate reuse of documented Nuxt configuration and presentation composition | Explicit extension contracts, merge precedence and integration tests |
| Declarative organisational specialisation without duplicated domain ownership | Review of archetype schemas and required capability providers |
| Contract-based service composition and security boundaries remain authoritative | More edges to include in dependency and compatibility checks |

## Guardrails

- No direct imports of another capability's private implementation.
- No hidden transitive or circular requirements.
- No bypass of server-side authorisation, assurance, tenant boundaries or privacy restrictions.
- No duplicate source-of-truth for state or lifecycle.
- No mandatory dependency on an optional integration.
- No new extension feature required of completed capabilities.
- A new extension surface is a public contract and must be documented and verified before consumers rely on it.
