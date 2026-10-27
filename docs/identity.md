# Identity and Access

> **In short:** who someone is, what groups they belong to, and what they may do, as separate capabilities. Identity owns identities, groups, the hierarchy and memberships; Authentication owns sign-in and sessions; Profile owns personal data; Authorisation decides. Groups form a forest with one parent each, every person has a personal group, and nothing is inherited through the hierarchy unless a rule says so. This replaces the identity and authorisation architecture and the group model definition ([ADR-0003](decisions/ADR-0003-group-model-and-identity-first.md), [ADR-0005](decisions/ADR-0005-iam-suite.md)).

**Status:** Normative

## 1. Separate capabilities

| Capability | Owns |
|---|---|
| Authentication | Who has signed in, and the secure lifecycle of that session |
| Identity | Identities (opaque, with no personal data), groups, the hierarchy, tenants and memberships with their states |
| Profile | The personal data describing a person, and what each context may see of it, under the person's control |
| Authorisation | Whether an actor may perform an action on a resource |

They stay distinct in concept and contract; with `iam-integration` they form the IAM suite (ADR-0005).

## 2. Terms

A **group** is an Identity-owned context for zero or more people, formed for a purpose such as an organisation, club, committee, event or department; it is not the domain entity it represents. A **membership** is a separately managed association between an identity and a group, never a single `groupId` on a person. **Resource ownership** names the person or group controlling a resource; **creator provenance** records who created it. A **tenant** is an isolation boundary, not a group.

## 3. Personal groups

Every human identity has exactly one system-managed personal group, created atomically with the identity: an incomplete identity never becomes active. Its one permanent member is that person. It is never reassigned, transferred, joined by anyone else or deleted apart from the identity; any delegation or recovery needs a separately authorised and audited contract. Non-human identities need their own lifecycle decision.

## 4. Hierarchy

Each group has zero or one parent and any number of children, so the groups form a forest of rooted trees. A group is never its own ancestor; reparenting rejects cycles, and a second parent is refused. Cross-organisation collaboration and other associations are explicit domain relationships or direct memberships, never a second parent. The hierarchy records organisational relationships only: no access, membership or ownership is inherited in either direction by default.

## 5. Memberships

A person may belong directly to any number of groups, in any trees, besides their personal group. Each membership has explicit references and a lifecycle status (such as active, suspended or ended, with effective times where needed). Changes are authorised, attributable and audited, and ending one membership changes nothing else. Whether the hierarchy confers membership is an explicit, versioned policy, never inferred from parentage.

## 6. Ownership

Every domain resource has an explicit ownership and access model for its domain and isolation context. A resource may record an owning group and, separately, its creator; the creator's identifier never implies continuing access, so a company's report stays the company's when its author leaves. Capabilities own their data and refer to others by public identifiers, never by cross-schema keys ([ADR-0002](decisions/ADR-0002-composition-supplied-persistence-and-capability-owned-schemas.md)).

## 7. Authorisation

Authorisation considers the resource: `actor + action + resource + context → decision`, using active memberships, roles and permissions, explicit grants and domain policy. Global roles alone are never the only control where resources have owners or scopes, and membership alone grants nothing unrestricted. Any inheritance rule states its direction, scope, exceptions and revocation, with negative tests. Tenant and resource isolation are separate checks, and a client-supplied group or tenant identifier is never proof of authority.

Every protected operation is authorised on the server. Route middleware and hidden controls are for the experience only; row-level security is extra containment.

## 8. Departure and revocation

When a membership ends, every permission derived from it stops, including inherited or delegated grants that depend on it. Cached decisions, session claims, long-lived tokens and asynchronous projections are covered by a documented revocation consistency guarantee, failing closed for sensitive operations. A genuinely independent grant survives; data already exported cannot be retracted; retained records and audit provenance follow their owner's retention policy.

## 9. Sessions and administration

Sessions follow current secure-cookie practice, with rotation, revocation, expiry, reauthentication for sensitive operations and a view of active sessions, according to risk. Administrative privileges are scoped and least-privilege; application administration, platform administration and resource ownership are kept apart unless explicitly required.

## 10. Domain integration

Domain capabilities associate their entities with groups by public identifier. The domain owns its business meaning and lifecycle, Identity owns membership and hierarchy, and Authorisation owns decisions; no domain entity becomes a second source of truth for membership. Management screens are authorised views over these contracts and own no group state.

## 11. Acceptance

Tests cover: atomic personal-group provisioning and exactly one personal group; many direct memberships; zero or one parent and many children; rejecting cycles and second parents; no implicit inheritance; isolation between tenants and companies; a creator leaving without ownership moving; membership-derived revocation, including caches and sessions; independent grants surviving; and refusal of unauthorised membership and hierarchy changes. The baseline is the platform's security requirements ([Security](security.md)).

Changing an invariant here needs an accepted ADR, with its migration and compatibility assessed first.
