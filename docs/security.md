# Security

> **In short:** the platform's security baseline (OWASP ASVS 5.0 Level 2, with Level 3 where it matters), how the programme is organised, and the controls every capability and data store must meet. It replaces the security architecture and the data store security standard ([ADR-0006](decisions/ADR-0006-polyglot-persistence-and-data-store-security.md)).

**Status:** Normative

## 1. Assurance baseline

Security is part of the architecture and an ongoing engineering responsibility.

The platform SHALL use **OWASP ASVS 5.0 Level 2** as its minimum application-security target. Verification records MUST state the exact ASVS release and SHOULD use version-qualified requirement identifiers.

Security-sensitive capabilities and operations SHALL also apply relevant ASVS Level 3 requirements where they are technically and operationally appropriate. This includes authentication, authorisation, privileged administration, session management, tenant and resource isolation, secrets and key management, and highly sensitive data handling.

A consuming application MAY impose stronger requirements but MUST NOT silently weaken this baseline.

## 2. Security programme

The platform uses one coordinated security-control programme rather than treating each standard as a separate compliance project:

- NIST Cybersecurity Framework 2.0 provides the overarching Govern, Identify, Protect, Detect, Respond and Recover structure;
- NIST SP 800-218 Secure Software Development Framework informs secure-development practice;
- OWASP ASVS 5.0 provides the application-security verification baseline;
- NIST SP 800-63-4 provides digital-identity assurance guidance;
- NIST SP 800-53, ABAC and Zero Trust guidance MAY be used as design and control references;
- ISO/IEC 27001:2022, Cyber Essentials, NHS DSPT/DTAC and HITRUST MAY be mapped as assurance or future compliance frameworks when applicable.

Reference to a framework MUST NOT be represented as certification, formal compliance or regulatory applicability unless that status has actually been established.

## 3. Principles

- secure by default;
- least privilege;
- deny by default for protected resources;
- server-side authorisation;
- defence in depth;
- explicit trust boundaries;
- minimal attack surface;
- strong tenant and resource isolation;
- dependency and supply-chain hygiene;
- secrets never committed to repositories;
- auditable sensitive operations;
- safe failure without disclosure of sensitive internals;
- explicit risk treatment for deferred controls.

## 4. Digital identity

Systems with authenticated users SHOULD be designed toward NIST SP 800-63-4 AAL2 when the application's risk and identity model make that assurance level appropriate. If an application claims AAL2, it MUST meet all applicable AAL2 requirements rather than selecting only convenient controls.

Phishing-resistant authentication MUST be available for elevated-risk accounts and SHALL be required for privileged platform-administration access unless a documented risk treatment establishes an equivalent or stronger control.

The platform does not claim AAL3 merely because individual AAL3-style controls are adopted.

## 5. Web and session controls

Applications MUST implement applicable ASVS baseline controls. The architecture MUST support, as applicable, secure session cookies, CSRF protection, input validation, context-appropriate output encoding, Content Security Policy and security headers, abuse controls, contemporary password protection where passwords are supported, session expiry, rotation and server-side revocation, reauthentication or step-up authentication for sensitive operations, and protection against account enumeration and credential abuse.

Client-side access controls are user-experience mechanisms only and MUST NOT constitute the authoritative security boundary.

## 6. Authorisation and isolation

Every protected operation MUST receive authoritative server-side authorisation.

Multi-user and multi-tenant resources MUST clearly define ownership, tenancy and access rules. A tenant or resource identifier supplied by the client MUST NOT, by itself, grant access.

Database row-level security SHOULD be used where appropriate as an additional containment boundary, particularly for tenant or owner-isolated relational data, but MUST NOT replace application authorisation policy.

Isolation controls MUST have negative tests demonstrating that one principal or tenant cannot access another's protected resources.

## 7. Trust boundaries and threat modelling

Threat modelling MUST consider, as applicable, the browser and other untrusted input, the Nuxt and Nitro server, persistence, identity providers, third parties, CI/CD, source control, package supply chain, deployment platform, administrator boundaries and tenant boundaries.

Security-critical capabilities MUST maintain a threat model proportionate to their risk.

## 8. Secrets and cryptography

Secrets MUST be supplied through appropriate deployment or runtime secret management and MUST NOT be exposed through public client runtime configuration.

Cryptographic design MUST use contemporary, reviewed mechanisms and provider and platform primitives where suitable. Custom cryptographic algorithms or protocols MUST NOT be introduced.

Key ownership, rotation, revocation and recovery requirements MUST be explicit where application-managed cryptographic keys exist.

## 9. Logging and audit

Security-relevant events MUST be auditable at a level proportionate to risk without logging credentials, session secrets, unnecessary personal data or other prohibited sensitive values.

Authentication events, authorisation-policy changes, privileged actions and security-control changes SHOULD produce structured audit records.

## 10. Secure development and supply chain

Repositories and delivery pipelines SHOULD implement proportionate controls including protected default branches and pull-request workflows, dependency and vulnerability scanning, secret scanning, static analysis, SBOM generation where useful, security-header and configuration tests, dynamic web scanning where appropriate, automated authorisation and isolation tests and dependency update discipline.

Security verification MUST be part of normal development rather than a pre-release-only activity.

## 11. Control register and evidence

The ecosystem SHOULD maintain a unified security control register mapping implemented controls to applicable ASVS, NIST and other assurance-framework requirements. Each control SHOULD identify status, implementation and evidence, responsible capability, verification mechanism, known gaps and risk treatment.

## 12. Cost constraints and deferred controls

Cost may affect how or when a security control is implemented, but it MUST NOT silently weaken a required security standard.

A required control that cannot yet be implemented MUST be recorded as a known gap with its rationale, affected assets, data and capabilities, risk, compensating controls where available and intended treatment or acceptance decision.

Paid certification, commercial security products and external assurance are not baseline requirements unless required by risk, contract or law.

## 13. Conditional higher-assurance contexts

ISO/IEC 27001 certification, Cyber Essentials certification, NHS DSPT/DTAC, HITRUST, health-sector controls, independent penetration testing, specialist SIEM services or external KMS services become mandatory only when application risk, data classification, contractual obligation or applicable law requires them.

The architecture SHOULD avoid preventable design choices that would make later adoption of such controls disproportionately difficult.

Applications processing special-category, health, regulated or otherwise high-impact data MUST perform a separate applicability and risk assessment before such processing is introduced.

## 14. Incident response and recovery

Applications handling material user data SHOULD maintain proportionate incident-response and recovery procedures aligned with current NIST guidance.

Recovery requirements, backup strategy and restore testing MUST be explicit for persistent data whose loss would materially affect users or platform operation.

## 15. Verification

Security claims MUST be backed by evidence appropriate to the claim. Automated tests SHOULD be used wherever feasible and manual review MUST supplement automation where tooling cannot establish the required property.

Certification, penetration testing and external assurance MAY provide additional evidence but do not replace secure architecture and continuous verification.

## 16. Data stores

This section applies to every capability that stores data, in any kind of store, with [Privacy](privacy.md) §3 and §6.

### 16.1 Inventory

Each capability MUST document, in its contract, every store it uses:

| Field | Meaning |
|---|---|
| Kind | Relational, graph, document, search, object, cache, time-series or other |
| Port | The provider function and the client type the host supplies |
| Holds | The data it holds, and whether it is the system of record or a derived copy |
| Classification | Whether it holds personal data, secrets or credentials (normally none outside Profile and Authentication) |
| Isolation | How tenant and group isolation is enforced (§16.3) |
| Erasure | How erasure and anonymisation reach it (§16.6) |
| Rebuild | For a derived copy, how it is rebuilt from the system of record |

### 16.2 Connections and credentials

1. The host supplies every client. A capability MUST NOT create its own connection from environment variables or embed credentials.
2. Each capability SHOULD receive credentials limited to its own schema, keyspace, index, bucket or prefix. Where the store supports it, the credential MUST NOT be able to read another capability's data.
3. Connections MUST use TLS outside a single trusted host, with certificate verification enabled.
4. Credentials MUST be rotatable without a capability release.
5. Administrative credentials (migrations, index creation) SHOULD be separate from runtime credentials and unavailable to request handling.

### 16.3 Authorisation and isolation

1. No store is reachable from the browser. Pre-signed object URLs are the only exception: they MUST be issued after server-side authorisation, scoped to one object and one operation, and expire within 15 minutes.
2. Every record, document, node, index entry and object key MUST carry its tenant identifier and owning group identifier.
3. Every query MUST be filtered by the tenant resolved on the server for the request. The filter is applied in the capability's data-access code, which callers cannot bypass.
4. Where the store cannot evaluate the platform's policy, the query MUST be narrowed by the caller's visible scopes from Authorisation's scopes query, and results MUST be checked per item for permissions whose risk level is `high` or `critical`.
5. On PostgreSQL, tables with tenant- or group-isolated rows SHOULD use row-level security:
   - policies read the tenant and scopes from transaction-local settings set with `SET LOCAL` (or `set_config(..., true)`), never session-level settings that a pooled connection could carry to another request;
   - runtime roles MUST NOT own the tables and MUST NOT have `BYPASSRLS`;
   - row-level security is containment only and never replaces application authorisation.
6. Graph traversals MUST be bounded in depth and MUST NOT cross a tenant boundary. A relationship between nodes of different tenants is refused on write.
7. Search and vector indexes MUST apply the tenant and scope filter inside the query, not by post-filtering a top-N result, so that results neither leak nor silently drop authorised items.
8. Caches MUST key entries by tenant and by every input that affects the result, including the caller's scopes where results differ by caller. Authorisation decisions MUST NOT be cached beyond the consistency bound Authorisation's contract allows.
9. Isolation tests MUST show, for each store, that one tenant cannot read or infer another tenant's data.

### 16.4 Encryption and secrets

1. Stores SHOULD be encrypted at rest by the provider or the host.
2. Secrets and credentials MUST be stored only by the capability that owns them, hashed or encrypted as its threat model requires, and never copied to derived stores.
3. Application-managed keys follow §8: owner, rotation, revocation and recovery stated.

### 16.5 Integrity and events

1. Writes that other capabilities must learn about MUST be published through a transactional outbox in the system of record, so that the event exists if and only if the write committed.
2. Derived copies MUST be idempotent under redelivery and tolerate out-of-order events, using a version or timestamp from the system of record.
3. Events and logs carry opaque identifiers and a correlation identifier only, never personal data, secrets or query text containing them.

### 16.6 Privacy lifecycle

1. Erasure and anonymisation MUST reach every store, including derived copies, caches and search indexes, within the capability's documented period. Backups age out on a documented schedule.
2. Anonymisation of a person is by unlinking ([ADR-0005](decisions/ADR-0005-iam-suite.md)): stores outside Profile hold only opaque identifiers, so removing Profile's link is sufficient unless a capability documents personal data of its own.
3. Retention periods MUST be documented per store and enforced by the capability.
4. A tenant's data region MUST be honoured by every store holding that tenant's data, including derived copies and backups. A capability that cannot honour a region refuses to store that tenant's data rather than storing it elsewhere.

### 16.7 Verification

Each capability's threat model and control register MUST reference this standard and record, per store, the evidence for §16.2 to §16.6. Tests use disposable local instances of each store, as ADR-0002 requires for PostgreSQL.
