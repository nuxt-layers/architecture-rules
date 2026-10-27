# ADR-0006 — Polyglot Persistence and Data-Store Security

**Status:** Accepted  
**Date:** 2026-10-09

> **In short:** capabilities may use stores other than PostgreSQL behind ports, provided each piece of data has one owner and system of record, authorisation is decided before any store is queried, every store enforces tenant isolation, and personal data stays in Profile.

## Why

[ADR-0002](ADR-0002-composition-supplied-persistence-and-capability-owned-schemas.md) settles relational persistence: the host supplies a PostgreSQL connection, and each capability owns its schema and migrations. Some capabilities will be better served by other kinds of store: a graph database for relationship-heavy domains, a search index, an object store for files, a cache, or a time-series store for metrics. ADR-0002 does not say whether these are allowed, who owns them, or how authorisation and tenant isolation reach data that never passes through PostgreSQL.

Two risks follow if this is left open. A capability may adopt a store whose own access model (for example a search index queried directly from the browser, or a vector store with no tenant filter) bypasses the server-side authorisation the platform requires. And data copied into a secondary store may escape revocation, erasure and data-region rules that its owner applies to the primary copy.

## The decision

1. **Other stores are allowed, behind ports.** A capability MAY use a non-relational store when its access pattern justifies one. It declares a port for it, as ADR-0002 requires for PostgreSQL; the host supplies the client. ADR-0002's rules on ownership, failing closed, provider SDKs and credentials apply unchanged to every kind of store.
2. **One owner, one system of record.** Each piece of data has exactly one owning capability and one authoritative store. Other stores hold derived copies (projections, indexes, caches) that the owner can rebuild and must keep consistent with revocation and erasure.
3. **Authorisation is decided before the store is queried.** No store is queried directly from the browser, and no store's built-in access model replaces server-side authorisation. A capability querying a store that cannot evaluate the platform's policy narrows the query with the caller's **visible scopes**: the set of groups and tenant whose information the caller may read, obtained from Authorisation's scopes query for that permission. Results are still checked per item where the permission requires it.
4. **Tenant isolation is enforced in every store.** Every stored record or document carries its tenant identifier and owning group identifier. Every query is filtered by tenant, with the filter applied by the capability's data-access code, never by the caller.
5. **Defence in depth on PostgreSQL.** Where a capability holds tenant- or group-isolated rows, it SHOULD enable row-level security, set the request's tenant and scopes with `SET LOCAL` inside the transaction, and connect with a role that is not the table owner and cannot bypass row-level security.
6. **Personal data stays where Profile puts it.** Under [ADR-0005](ADR-0005-iam-suite.md), stores outside Profile hold opaque identifiers, never personal data, unless a capability's documented purpose requires it and its erasure procedure covers every store it uses. One such purpose is settled here, amending ADR-0005 §1:
   - **Sign-in identifiers stay in Authentication.** An email address or other identifier used to sign in is a credential identifier, owned and verified by Authentication, so that signing in never depends on Profile being available. Authentication uses it only for signing in, account recovery and security notices.
   - **Contact details stay in Profile.** Profile may hold the same address as a contact detail for a different purpose. The two copies are independent: changing one does not change the other unless the person asks for both to change.
   - **Erasure covers both.** Account closure and erasure requests reach the sign-in identifier in Authentication as well as Profile's record; the process is specified in `iam-integration`.
7. **Data region follows the tenant.** A tenant's data region, once set, applies to every store holding that tenant's data, including derived copies and backups.

The concrete controls are in [Security](../security.md) §16.

## What follows

| Gains | Costs |
|---|---|
| Capabilities can choose the store that suits them without weakening authorisation, isolation or privacy | Authorisation must provide a scopes query, and hosts must make it available to capabilities that use non-relational stores |
| Revocation, erasure and data-region rules have one owner for each piece of data | Each derived copy needs a rebuild and erasure path, with tests |
| Authorisation stays the single source of access decisions, including for stores that cannot run its policy | Hosts operate more kinds of infrastructure, each needing credentials, TLS, backups and monitoring |

## Guardrails

- A capability that adds a store documents it in its `CONTRACT.md` and threat model, with the store's port, what it holds, its system of record and its erasure path.
- Isolation tests ([Security](../security.md) §6) cover every store a capability uses, not only PostgreSQL.
- Changes to these rules require a new ADR.
