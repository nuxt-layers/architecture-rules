# ADR-0017 — Tenant Hierarchy and Accountable Digital Control

**Status:** Accepted  
**Date:** 2026-11-16  
**Amends:** ADR-0003 and ADR-0005 on **tenant** governance; preserves their **group** invariants.  
**Review evidence:** [Blueprint PR #39](https://github.com/nuxt-layers/architecture-blueprint-review/pull/39), including `proposals/organisational-tenancy-and-governance.md` and `reviews/organisational-ownership-and-recovery.md` on merged `master`.

## Context

One person can establish several businesses, join a club and a council, and administer subsidiaries without sharing their confidential records. A contractor can create a digital company representation while the legitimate organisation remains entitled to direct its digital operations. Current [Identity and Access](../identity.md) and ADR-0003 establish a single-parent **group** forest, not a tenant parentage contract. ADR-0005 governs group founding/ownership, not an independently verified corporate controller.

## Decision

1. **Distinct scopes.** An installation is the deployment and composition, not a tenant. An Identity **tenant** is a security/data-isolation boundary. An Identity **group** is an organisational context with memberships. A real-world legal/business entity is a separate domain concept. None is automatically one-to-one with another. A person keeps one Identity principal/personal group and can have independently authorised memberships/roles across many tenants.
2. **Tenant tree.** Identity owns a separate, acyclic **tenant parent relationship**: at most one direct parent and any number of children, with explicit lifecycle/provenance. It is **not** the ADR-0003 group forest, cannot introduce a second parent to a group and cannot place a child tenant's data inside a parent's storage scope by implication. Any reparenting requires protected authorisation, compatibility/security checks and an attributable change. Root tenants have no parent.
3. **No implicit power.** Tenant parentage, platform-host ownership, group parentage, creator provenance, legal title, a subscription and a purchased entitlement grant **no automatic resource access, group membership, payer liability or cross-tenant rights**. Authorisation must decide each action/resource/context using independently approved, scoped, expiring/revocable delegation where appropriate. Parent administration can cover tenant lifecycle and approved services without exposing child customers, accounts or personal records.
4. **Accountable digital control.** Every **active/operational** tenant SHALL have at least one active, verified human digital governance owner. Multiple human owners and jointly governed action are supported. No non-human/AI identity counts towards this floor. Founding/transfer/departure and delegation are governed Identity-owned facts, separate from creator, director, shareholder, real-world controlling party, employee, administrator and commercial payer. Creator attribution never becomes irrevocable control of business data. Group founding-owner and last-owner requirements under ADR-0005 remain unchanged.
5. **Recovery and succession.** Normal changes MUST prevent voluntary removal of a tenant's last verified human owner. If all such owners become unavailable (death, compromise or loss of standing), the tenant enters an explicitly restricted **recovery/exception state**, never an invented new verified owner; existing record retention and permitted continuity remain governed. Recovery, succession, contested transfers and legal-control challenges require independently evidenced authority, appropriate qualified review, strong/recency authentication, separate authorisation and independent approval/delay proportionate to risk. No participant may weaken the change policy and immediately exploit its previous authority. Invalid/uncertain authority is **held**, not assumed valid.
6. **Legal versus digital authority.** Identity owns the **digital control relationship and lifecycle**, but MUST NOT independently assert legal-company control solely from a role label, claimed directorship or shareholding. Verification of governing instruments, legal status and disputes belongs to a separately justified authority/Legal Compliance interface; its definitive data owner remains an explicit open design question. Verified evidence may authorise a digital-control change without contractor cooperation under a protected, auditable procedure, not unilateral platform impersonation.
7. **Digital ownership does not transfer source data.** Data-owning capabilities preserve organisational resource ownership, provenance, privacy, retention and export rules even when creator/administrator/tenant owner changes. Legitimately authorised, privacy-constrained export and migration MUST have a route not vetoable solely by the original creator or a delegated contractor. Export availability does not waive legal holds or third-party rights.
8. **Safe bootstrap and self-service.** Installation bootstraps a verified human platform administrator and a separately scoped operator business organisation/tenant. Routine permanent 'god-user' access or unrestricted customer-data reads are prohibited; exceptional operator recovery is attributable, constrained and independently auditable. Free personal provisioning and paid, policy-authorised organisational onboarding MAY be exposed via a secure host composition. The current Identity privileged tenant/root-group provisioning API remains operator-only; browser clients MUST NOT receive its migration credentials or invoke privileged internal provisioning.
9. **Policies and change evidence.** Mandatory security floors are established by accepted rules and enforced by owners, not tenant-editable switches. Configurable governance approvals are versioned, bounded and cannot weaken those floors (ADR-0018). Protected tenant/group/owner changes are attributable, publish authoritative events through capability-owned durable outboxes when required, and supply Audit evidence without reowning it.
10. **Independence and composition.** Identity owns tenant and group authoritative data, Authentication credentials/assurance, Authorisation decisions, Profile personal data and `iam-integration` suite coordination. Host/workflow orchestration MUST use public contracts and preserve ADR-0012's composition limits and ADR-0016's directed acyclic dependency graph; this ADR creates no standalone `tenancy`, `business` or legal-entity capability.

## Implementation/contract handover (not claimed implemented)

Identity's public contract must eventually document create/provision/request, describe, parent/reparent, owners/transfer, recovery/exception, revoke, close and export operations with trusted subject/tenant context, state/effective version, refusal codes and audit/outbox semantics. Cross-tenant delegated authority belongs to Authorisation's documented decision/grant boundary. The host resolves effective tenant from trusted context and never treats browser-submitted IDs as permissions. IAM integration handles verified workflow handoffs without persisting duplicate principals/tenants.

**Do not infer these APIs already exist.** Current `nuxt-layers/identity/CONTRACT.md` exposes privileged host tenant/root-group provisioning and group ownership, not the tenant tree and verified-tenant-owner lifecycle required by this accepted decision.

## Required conformance evidence

- Distinct group and tenant trees; reject cycles, illegal reparenting and unauthorised tenant creation.
- One person in several businesses/groups and independent tenants, no cross-tenant leakage or parent access by default.
- New active tenant has genuine verified human owner; removing last owner refused, exceptional loss becomes restricted recovery.
- Two owners and high-risk owner change; prevent self-approval and policy weaken-then-transfer.
- Fraudulent legal claimant refused/held; independently evidenced legitimate controller can contest hostile contractor without impersonation.
- Parent admin can administer permitted subscription/tenant facts without reading child private records; changing payer does not move data or ownership.
- Creator/contractor removed: no remaining privileged access, no veto on authorised export; audit preserved, legal holds enforced.
- Bootstrapping has no ordinary superuser bypass; provisioning failure and duplicate requests are safe.

## Consequences, compatibility and migration

Identity, Authorisation and `iam-integration` require reviewed contract/version changes before this can be operationally advertised. Existing tenants and groups MUST NOT be assigned invented verified human owners; any migration must reconcile evidence and place uncertain cases in a reviewed exception state. Existing ACLs, membership and parent-group semantics do not silently change. ADR-0003/0005 group invariants remain binding. No implementation change is required merely because this proposal is merged.

## Guardrails and unresolved decisions

- Qualified verification/adjudication owner for real-world controlling entity and governing instruments.
- Authority and duration for disputed-ownership holds, notifications and appeal.
- Tenant-parent consent and change of parent under business sale or termination.
- Minimal recovery service availability, data portability per domain and personal-data constraints.
- Concrete tenant-owner identifier and secure provisioning public contract without privilege escalation.

Resolve these in capability design and later approved contract changes; they do not justify inventing a universal legal-owner engine.
