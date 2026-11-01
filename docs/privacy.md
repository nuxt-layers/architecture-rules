# Privacy

> **In short:** data protection is designed in, not added as a banner. Capabilities collect only what their purpose needs, classify every storage and tracking technology, ask for consent before anything that needs it, and support access, export, correction, deletion and retention without manual searching.

**Status:** Normative

## 1. Principle

Privacy and data protection are part of the architecture rules. Compliance MUST NOT depend only on a cookie banner or privacy-policy page.

## 2. Scope

The architecture governs cookies, browser storage and similar technologies; telemetry and analytics; third-party integrations; what personal data is collected and why; consent and preferences, where consent is the basis; retention; access and export; deletion; and a record of people's privacy choices.

## 3. Data minimisation

Capabilities MUST collect and retain only data justified by their documented purposes. Optional telemetry SHOULD be absent by default in the initial platform unless a defined need justifies it.

## 4. Classifying storage and tracking

Client storage and access mechanisms MUST be inventoried and classified by purpose and necessity. Mechanisms requiring consent MUST NOT be activated before valid consent.

Technically necessary mechanisms MUST still be documented transparently.

## 5. Storage register

Every capability that creates or causes persistent storage MUST maintain a root `STORAGE-REGISTER.md`, covering server-side stores, cookies, browser storage, caches and third-party storage. Record each mechanism's name, owner, purpose, exact data or fields, personal-data classification, storage technology, scope and isolation, retention/expiry and deletion, access and security controls, whether it is technically essential for core operation (with justification and behaviour when unavailable), and consent-relevant purpose and necessity. Technical necessity does not automatically establish a legal consent exemption. For cookies include domain, path, first-/third-party status, `Secure`, `HttpOnly` and `SameSite` where applicable. Clearly distinguish implemented storage from proposals; mark unknown properties as unverified rather than guessing.

The capability owns the factual inventory and keeps it in step with implementation. Its standalone `CONTRACT.md` includes the storage information and host obligations needed for correct integration without requiring the consumer to read the register. The consuming application determines jurisdiction-specific notices, exemptions, consent and withdrawal. The register does not itself establish legal compliance; storage that requires consent MUST remain inactive until valid consent has been obtained.

A genuinely stateless library does not need a register; its contract states that it persists no data.

## 6. Jurisdiction

Reusable capability contracts SHOULD avoid hard-coding the rules of one jurisdiction. Applications MAY provide policies or adapters for the jurisdictions they need to support.

The first applications are designed for UK data-protection and electronic-communications law, keeping room for other jurisdictions.

## 7. User rights and lifecycle

The architecture SHOULD support access and export, correction, erasure, retention rules and account closure, without anyone having to search for a person's data by hand.

## 8. Security relationship

Privacy and security overlap, but they are not the same. Authorisation controls who can access data. Privacy also asks whether the data should be collected, processed, retained, disclosed or transferred at all.
