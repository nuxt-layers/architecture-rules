# ADR-0007 — Break-Glass Emergency Access

**Status:** Accepted  
**Date:** 2026-10-09  
**Amended:** 2026-10-26: brought in line with ADR-0010 and later decisions

> **In short:** a single-operator platform may keep break-glass accounts: passkey-only, with no standing privileges, able only to suspend and to appoint an owner to an orphaned group, alerting everyone and reviewed by someone else after every use.

## Why

[ADR-0005](ADR-0005-iam-suite.md) makes the IAM suite's governance deliberately hard to bypass: no impersonation, a two-person rule for `high` and `critical` changes, and no emergency bypass of approvals. During a security incident, a platform operator's suspension of an identity is approved by a second operator.

That rule assumes at least two platform operators. The platform is expected to run, at least initially, with a single operator. In an incident (a compromised owner account, an attacker escalating inside a group), a single operator could not act until a published delay ran out. Leaving that gap open is a greater risk than a tightly constrained emergency path.

Relaxing a security control requires a documented risk treatment. This ADR is that treatment.

## The decision

1. **Break-glass accounts exist only as a constrained emergency path.** A host MAY provision break-glass accounts. Where a deployment has fewer than two platform operators, it SHOULD provision at least one.
2. **They hold no standing privileges.** A break-glass account is an ordinary identity with no roles until it is used. It acts under its own identity, never as another person (ADR-0005 §2.2 is unchanged).
3. **Passkeys only.** Break-glass accounts authenticate with phishing-resistant passkeys held offline, never used for day-to-day work, and never with passwords or recovery codes.
4. **They do two things only:**
   - suspend an identity or membership;
   - appoint an owner to an orphaned group (the recovery process).

   They cannot change roles, grants, policy, group structure or data, and cannot read any group's information.
5. **The second operator's approval is replaced by after-the-fact review.** A break-glass action takes effect at once. Every use:
   - raises an alert to every platform operator and every owner of the affected group or tenant;
   - is recorded with a reason code and a correlation identifier;
   - opens a mandatory review that must be closed by a person other than the user of the account.
6. **The account is spent after use.** After every use, its passkey is rotated and the account returns to no privileges.

## What follows

| Gains | Costs |
|---|---|
| A single-operator platform can contain an incident immediately | Hosts must provision, store and periodically test break-glass passkeys |
| The path is narrow: it can stop harm (suspend) and restore governance (appoint an owner), but cannot grant, change or read anything else | Authentication must recognise break-glass accounts and enforce passkey-only sign-in for them |
| Every use is visible and reviewed | Identity and Authorisation must expose the two break-glass actions and the review record |

## Guardrails

- Adding any capability to break-glass accounts beyond §4 requires a new ADR.
- The emergency process is specified in `nuxt-layers/iam-integration` (approvals, pausing and suspension, recovery).
- Break-glass use is a test case in each affected capability's security tests: allowed for the two actions, refused for everything else.
