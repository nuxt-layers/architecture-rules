# ADR-0009 — Headless Capability Layers

**Status:** Accepted  
**Date:** 2026-10-21  
**Amended and accepted:** 2026-10-22: no screen exceptions for sign-in or step-up; client helpers and protocol endpoints named; wording keys, theme authority and person names placed; screens removed only once replaced  
**Amended:** 2026-10-22: `ui` offers styled, customisable components; kits are named `<capability>-ui`; composition kits and the host's part defined  
**Amended:** 2026-10-25: a layer's core stays headless, but the layer may ship an optional presentation part for its own screens; kits become optional; released layers keep their screens

> **In short:** a capability layer's core does the work and never depends on screens. A layer may also ship its own screens as an optional, separable part, drawn only with Theme Manager's vocabulary, which a host uses, overrides or switches off. Whoever composes it decides.

## Why

Several of our layers have grown their own screens, styles and wording. A host that composes them inherits several opinions about how things should look, and every layer repeats the same presentation work.

A layer's real value is what it knows and guarantees, such as a document's lifecycle or a person's consent. How that is shown is a separate concern, and whoever composes the layer must be able to decide it.

The 2026-10-22 decision went further and removed every screen from every layer, moving them into separate kit repositories. Composing the IAM suite into Westray-Papa-Westray showed what that costs: each application redraws and retests the same sign-in, account, security and profile screens, or waits for four kits and `ui` to be built and released, while the layers' own screens already work, follow the theme and can be switched off. The amendment keeps what made the decision sound (a core that never depends on screens, safety on the server, a host that can always draw its own) and drops the cost of moving every screen out.

## The decision

Every capability layer, whether foundation, platform or domain, has a **headless core**, and may have an **optional presentation part**.

| The headless core offers | The core never offers or depends on |
|---|---|
| **Contracts:** types, schemas and pure helpers | Pages, routes, layouts or components |
| **Server functions:** one clear task each | Styling or class maps |
| **An HTTP API**, which the host may switch off | Display wording or its translations |
| **Protocol endpoints** a standard requires, such as a sign-in provider's callback or an emailed link that redirects; they answer a redirect or data, never a page | |
| **Events** it publishes, and **ports** it asks the host to fill | |
| **Data formats** such as feeds, export files and a theme's CSS | |
| **A typed client**, holding data, actions, navigation guards and browser ceremonies (such as a passkey prompt); guards take their destinations from configuration | |
| **Codes and keys:** error codes and `messageKey`s, with their parameters | |

### The optional presentation part

A layer may ship pages and components for its own capability, for one audience at a time (people themselves, tenant administrators or platform operators, never mixing self-service with administration). When it does:

- **It is separable.** It depends only on the core's contract and typed client, never the reverse. The host can switch it off entirely (`<layer>: { presentation: false }`), keep the components without the pages, or move the pages.
- **It is styled only with Theme Manager's vocabulary** (`SemanticPresentationTheme`): its class names, no styles, tokens or theme handling of its own, and no package dependency on Theme Manager. Without Theme Manager the screens work, unstyled.
- **Its wording is a message catalogue** keyed by `messageKey`, in the layer's default locale, which the host overrides or translates in its own configuration. The keys are part of the contract.
- **It is proved by the layer's tests,** including accessibility in light and dark mode, against Theme Manager's default theme.
- **It takes other capabilities' data from the host,** such as a person's name, through a component the host can override; it never reaches another capability itself.

### Theme Manager

Theme Manager is headless too. Its themes, tokens, contrast checks and preferences are services; its CSS is a data format, and applying the effective theme to the page is delivering that format, not drawing. It stays the **only authority** for the effective theme and its token vocabulary. Layers' presentation parts and `ui` consume that vocabulary and never define, validate or apply a theme of their own.

## Where presentation lives

| Who | Does what |
|---|---|
| **A layer's presentation part** | Its own capability's standard screens, as above. The default for screens every application needs, such as sign-in, account, security and profile |
| **`nuxt-layers/ui`** | Generic, fully styled components on Theme Manager's tokens, which layers' presentation parts, kits and applications may share. No component knows a capability. Nothing is required to move into it |
| **Composition kits** | One journey across several capabilities, such as managing users; created when a second application needs it. No data or services of their own |
| **Applications** | Compose capabilities and use, override or replace their screens; own everything specific to the product, and every screen no layer offers |

Capability kits (`<capability>-ui`) are no longer required. One is created only if a capability's screens must be released apart from the capability, and the case is argued in the layer's contract.

Every screen, wherever it lives:

- takes the tenant, the signed-in person and other capabilities' data from the application, never from a request or a lookup of its own;
- uses only the capability's contract, typed client and HTTP API, never its storage or private code ([ADR-0001](ADR-0001-capability-repositories-and-composition-shells.md));
- follows the [presentation, internationalisation and accessibility baseline](../presentation.md).

## Safety stays with the data

Presentation never holds a guarantee.

- **Risky content is returned in a safe form.** Rich text comes back as structure that cannot be shown unsafely, never as markup.
- **Rules are enforced by the core's functions.** Permissions, step-up, consent, rate limits and visibility hold whichever screen calls them, the layer's own or an application's.
- **What a screen must still do is written down.** Each layer's contract has a *Headless surface* section listing it, such as no framing or caching of administration screens, or the right `autocomplete` on credential fields, so an application that draws its own screens can meet it.

## Adopting it

| Layer | Adopts it |
|---|---|
| New, or before 1.0 | A headless core from the start; a presentation part only where the layer's screens are standard. `publisher` stays fully headless: its editing comes from `content-editor` through its editor slot |
| Released, with screens | Keeps them as its presentation part. If any screen breaks a rule above, it is fixed in a minor release; nothing is deprecated for being a screen |

The headless migration plan ([Resources](../../RESOURCES.md): *Headless migration v0.1*) is superseded by this amendment, except for `ui`'s first release, which still removes `ui`'s competing tokens and theme handling; that release belongs to `ui`'s own plan.

## What follows

| Gains | Costs |
|---|---|
| A composed application looks and reads as one product, through the shared theme vocabulary | A layer with a presentation part releases for screen fixes as well as for its services |
| Standard screens are built and tested once, in the layer, for every application | A layer's screens follow its own layout; an application wanting another draws its own |
| Applications compose services without competing routes or styles: every presentation part can be moved or switched off | Wording lives in each layer's catalogue, not in one place per application, unless the application overrides it |
| Any front end can still use a capability: console, website, app or another service | Two places can draw a capability's screens; the contract's *Headless surface* keeps them equally safe |

## Guardrails

- A capability's contract lists its public surface, its presentation part (pages, components, message keys) and its *Headless surface* duties.
- A test fails if the core imports the presentation part, or if the presentation part uses styles, tokens or classes outside Theme Manager's vocabulary.
- With `presentation: false` the layer registers no page, component, route or style, and its core works unchanged; a test proves it.
- A capability's codes and `messageKey`s are part of its contract: renaming or removing one is a breaking change.
- Screens, wherever they live, never reach a capability's storage or private code (ADR-0001).
