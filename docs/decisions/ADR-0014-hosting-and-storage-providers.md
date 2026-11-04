# ADR-0014 — Hosting and Storage Providers

**Status:** Accepted  
**Date:** 2026-11-04

> **In short:** hosting and storage providers may be outside the UK when UK data-protection law permits the transfer, provided each tenant's data region is honoured and each provider's security is verified independently before use and re-checked yearly.

## Why

The first applications are designed for UK law ([Privacy](../privacy.md) §6), and a tenant's data region applies to every store ([ADR-0006](ADR-0006-polyglot-persistence-and-data-store-security.md) §7). Neither says whether a provider may host or store data outside the UK, nor what assurance a provider must give. Requiring UK-only providers would narrow the choice and raise costs without improving protection; accepting any provider on its word would leave personal data with no verified safeguards.

## The decision

1. **Providers may be outside the UK where UK law permits the transfer.** A hosting or storage provider may process or store data outside the UK only where UK data-protection law permits the transfer:
   - to a country covered by UK adequacy regulations, the EU and EEA included; or
   - otherwise, under the UK international data transfer agreement (IDTA) or the UK addendum to the EU standard contractual clauses, with a transfer risk assessment.
2. **Data region is recorded and honoured.** Each tenant's data region is recorded and honoured by every store holding that tenant's data, including derived copies and backups ([ADR-0006](ADR-0006-polyglot-persistence-and-data-store-security.md) §7, [Security](../security.md) §16.6).
3. **Provider security is verified independently before use.** A provider is used only when the following are confirmed:

   | Requirement | Evidence |
   |---|---|
   | Independent assurance | A current ISO/IEC 27001 certificate or SOC 2 Type II report covering the service and region used |
   | Processing terms | A data processing agreement and a published sub-processor list |
   | Encryption | In transit and at rest |
   | Breach notification | Without undue delay, in time for the application to meet UK GDPR's 72-hour period |
   | Deletion | On request and at contract end |

4. **Each application records its providers.** An application records, in its `RESOURCES.md`, each provider's region, assurance evidence and the date it was checked. It re-checks each provider yearly and whenever the provider or the service used changes.
5. **Capabilities are unaffected.** Capabilities stay provider-agnostic behind ports ([ADR-0002](ADR-0002-composition-supplied-persistence-and-capability-owned-schemas.md), [ADR-0006](ADR-0006-polyglot-persistence-and-data-store-security.md)). Choosing and verifying a provider is the application's job; nothing changes for capabilities.

## Adopting it

Each application records its current providers in its `RESOURCES.md` the next time its hosting or storage changes, and before any new provider is used. A provider that fails a check is not used until it passes.

## What follows

| Gains | Costs |
|---|---|
| A wider, cheaper choice of providers within UK law | Each application checks and records its providers, and re-checks them yearly |
| Assurance rests on independent evidence, not a provider's word | Transfers outside adequacy need an IDTA or addendum and a transfer risk assessment |
| Capabilities and their ports stay unchanged | Providers without ISO/IEC 27001 or SOC 2 Type II cannot be used |

## Guardrails

- This decision covers providers' assurance. It does not make the platform's own certification mandatory ([Security](../security.md) §13).
- A store that cannot honour a tenant's data region refuses that tenant's data rather than storing it elsewhere ([Security](../security.md) §16.6).
- Changes to these rules require a new ADR.
