# ADR-0005 — Identity and Access Management Suite

**Status:** Accepted  
**Date:** 2026-10-09  
**Amended:** 2026-10-23: `iam-integration`'s documents pinned (§3)

> **In short:** five capabilities form the IAM suite, meeting only through contracts and ports: Authentication, Identity, Profile, Authorisation and `iam-integration`. Ten suite-wide invariants, from self-sovereignty to durable events, bind them all.

## Why

Authentication and Authorisation are implemented; Identity is next ([ADR-0003](ADR-0003-group-model-and-identity-first.md)). Their boundaries were drawn one capability at a time. Before Identity is designed, the capabilities that together answer "who is this, what are they a member of, and what may they do?" need one agreed set of boundaries, one place for the processes that span them, and a small number of suite-wide invariants that no single member can enforce alone.

The earlier catalogue placed "baseline identity/profile information" in Identity. That would put personal data beside the membership graph that every decision reads, widening the data most capabilities must handle.

## The decision

### 1. Members

The Identity and Access Management (IAM) suite has five members. Each is an independent capability repository; none imports another.

| Member | Owns | Never owns |
|---|---|---|
| `authentication` | Credentials, sign-in identifiers, authenticated sessions, step-up and reauthentication | Groups, personal data other than sign-in identifiers ([ADR-0006](ADR-0006-polyglot-persistence-and-data-store-security.md) §6), access decisions |
| `identity` | Opaque identity identifiers and their lifecycle; personal groups; groups, the single-parent hierarchy and tenants; memberships and their states; group governance settings | Names, contact details or any other personal data; credentials; access decisions |
| `profile` | The personal data that describes a person, keyed by an opaque Identity identifier; what each context may see; the person's access, correction, export and erasure requests; anonymisation by unlinking | Credentials, sign-in identifiers, memberships, access decisions |
| `authorisation` | Roles, assignments, grants and server-side access decisions | Groups, memberships, personal data |
| `iam-integration` | The suite's architecture, its cross-capability processes, and the adapters and single composition call that wire the members ([ADR-0012](ADR-0012-trivial-composition.md) §3) | Any of the above data |

Members meet only through public contracts and host-supplied ports. `iam-integration` is distinct from the general-purpose `integrations` hub.

### 2. Suite-wide invariants

1. **Self-sovereignty.** A person controls their own personal group and account. No other principal, including a tenant or platform administrator, can act inside a personal group except through a separately authorised and audited recovery process.
2. **No "act as".** No member provides impersonation. Support access is a scoped, time-limited grant to the supporter's own identity, recorded as such.
3. **Founding owner.** Creating a group makes the creator its first owner. A group always has at least one owner; removing the last owner is refused.
4. **Two-person rule by risk.** A `high` or `critical` change to roles, ownership or governance requires approval by a principal other than the requester. A group may raise this requirement, never lower it. Where a group has one owner, approval comes from an owner of the parent group or tenant, or from a published time delay the owner cannot shorten.
5. **Membership states.** `active`; `paused` (chosen by the member); `suspended` (imposed by others); `ended`. Only `active` memberships confer access. Roles held under a paused or suspended membership are kept but inactive.
6. **Pausing.** A member may pause one membership or their whole account. A paused member can still view the group's information where their roles allow, but is hidden from other members and receives no notifications or assignments. A group may restrict pausing within the group; it cannot prevent a person pausing their whole account.
7. **Departure.** On leaving a group, all access derived from that membership ends ([Identity and Access](../identity.md)). Each group sets what happens to the leaver's attribution in its information: keep the name, pseudonymise or anonymise. The leaver may always require anonymisation, within the law.
8. **Anonymisation by unlinking.** Members store opaque identifiers, never personal data. Anonymisation removes the link from an identifier to Profile's record; it does not rewrite other capabilities' data.
9. **Revocation reaches sessions.** Suspending, pausing an account or closing an identity revokes its sessions in Authentication. Membership changes reach Authorisation within its directory consistency bound.
10. **Durable events.** Lifecycle changes that other members must act on are published through a transactional outbox, carrying opaque identifiers and correlation identifiers only.

### 3. Where the detail lives

Following [ADR-0004](ADR-0004-documentation-placement.md), the suite architecture, the state models and the cross-capability processes (provisioning, joining and leaving, pausing, suspension, approval, account closure with a grace period, data-subject requests, recovery) are specified in `nuxt-layers/iam-integration`: its architecture, states and processes ([Resources](../../RESOURCES.md): *IAM suite architecture*). Each member's own contract documents its part.

## What follows

| Gains | Costs |
|---|---|
| Identity stays small: it holds a graph of opaque identifiers, which most capabilities can read without handling personal data | One more capability (`profile`) and one more repository (`iam-integration`) |
| Erasure and anonymisation have one owner (Profile) and one mechanism (unlinking) | Authorisation's contract must add the `paused` membership status (treated like `suspended` for decisions); this is a contract version change |
| Suite-wide rules are written once and enforced by each member against its own data | Approval flows need durable pending state, owned by Identity for governance changes and by Authorisation for role and grant changes |
|  | Displaying a member's name requires a Profile lookup, subject to Profile's disclosure rules |

## Guardrails

- A member that stores personal data outside Profile, or imports another member, fails review.
- Changes to these invariants require a new ADR.
- Tenant provisioning remains a candidate `tenancy-service`; until it exists, Identity holds tenants.
