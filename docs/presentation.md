# Presentation, Internationalisation and Accessibility

> **In short:** every screen is built from Theme Manager's semantic tokens, works in any language through BCP 47, CLDR and ECMA-402, meets WCAG 2.2 AA in light and dark mode, and uses semantic HTML. Preferences never break accessibility, and every screen passes the checklist in §8.

**Status:** Normative

## 1. Purpose

Presentation quality, internationalisation, accessibility and semantic web support are platform-wide concerns. They MUST be designed in from the start rather than added after a feature is complete.

## 2. Presentation system

Reusable presentation capabilities MUST use a shared design system built from semantic design tokens. Applications MUST NOT rely on scattered hard-coded presentation values.

Where applicable, the presentation system MUST support semantic colour roles; typography and font preferences; spacing, sizing, density and radius tokens; responsive layouts; light, dark and high-contrast modes; operating-system preferences; user-selectable presentation preferences; and persistence through an appropriate settings contract.

User customisation MUST NOT break mandatory accessibility requirements. Theme and preference implementations MUST constrain, validate, adjust or reject combinations when needed to preserve contrast, readability, visible focus, reflow and usable interaction.

## 3. Internationalisation

Internationalisation is a standard platform capability, built on these standards:

- **BCP 47** language tags identify every locale (for example `en-GB`) in configuration, routes, `lang` attributes and message catalogues;
- **Unicode CLDR** locale data, through **ECMA-402** (the JavaScript `Intl` API), formats dates, times, numbers, currencies, plurals, lists and sorting; nothing is formatted by hand;
- the **W3C Internationalization** guidance applies to markup: language and direction declared, and right-to-left scripts not prevented.

Applications and reusable capabilities MUST keep user-facing text out of domain logic and make that text localisable. They MUST declare the document language correctly and support locale-aware formatting. Capabilities publish codes and `messageKey`s, never wording ([ADR-0009](decisions/ADR-0009-headless-capability-layers.md)). Each presentation part, `ui` and any kit MUST own the message catalogue for what it draws; the application MAY override any message and MUST NOT be forced to maintain one large translation catalogue.

Locale-aware routing and translated metadata SHOULD be supported where public content requires them. An application MAY initially enable only one locale, but its architecture MUST NOT unnecessarily prevent additional locales.

## 4. Accessibility

WCAG 2.2 Level AA is the minimum accessibility engineering target for web presentation produced by the ecosystem.

Accessibility is a component and capability acceptance criterion. Implementations MUST account for semantic HTML, keyboard operation, visible and predictable focus, accessible names and labels, contrast, text resizing and reflow, target size, reduced motion, status and error messages, language metadata and assistive technology.

ARIA MUST supplement native semantics where necessary and MUST NOT replace suitable native HTML semantics without justification.

Automated accessibility testing SHOULD form part of CI, but MUST NOT be treated as sufficient evidence of conformance without appropriate manual verification.

## 5. Semantic web and structured data

Public-facing capabilities MUST prefer semantic HTML and meaningful document structure.

When structured data is useful for public content, applications SHOULD use appropriate Schema.org vocabulary in a machine-readable form such as JSON-LD. Structured data MUST truthfully describe the resource and MUST NOT invent information purely for search presentation.

Metadata, canonical URLs, locale alternates and structured data SHOULD be generated from authoritative domain data rather than duplicated manually where practical.

## 6. Layer responsibilities

`ui` owns reusable presentation primitives and their accessibility behaviour; each layer's presentation part owns its own screens ([ADR-0009](decisions/ADR-0009-headless-capability-layers.md)). Theme Manager alone owns the effective theme and its token vocabulary.

Internationalisation, semantic metadata and user-preference persistence MAY be separately bounded capabilities or integrations where their responsibilities justify separation. Their public contracts MUST remain independent of any single replaceable implementation library.

Domain capabilities SHOULD supply semantic content and metadata inputs without owning global presentation infrastructure.

## 7. Verification

Reusable UI components SHOULD have automated checks for accessibility-relevant behaviour where feasible. Consuming applications MUST verify accessibility at the composed-page and workflow level because component-level compliance alone cannot establish application-level WCAG conformance.

Internationalisation tests SHOULD include missing-message behaviour and locale-sensitive formatting. Structured data SHOULD be schema-validated where practical.

## 8. Screen checklist

Every screen over a capability, whether a layer's presentation part, `ui`, a kit or an application draws it, MUST meet this list, and the drawing repository's tests MUST prove it.

| Check | Proved by |
|---|---|
| Each item in the capability's *Headless surface* section | The drawing repository's tests, against that contract version |
| No serious or critical axe finding, in light and dark mode | Browser tests per page |
| Keyboard operation, visible focus and one `<main>` per screen | Browser tests per workflow |
| Administration and credential screens are never framed, cached or sent as a Referer | A test of the response headers |
| Rich text drawn only from safe trees, never as HTML | `ui`'s renderer tests; a search for raw HTML insertion |
| Every message has a key; a missing one fails visibly in development | Internationalisation tests |
| No name or contact detail in a document title or URL | Browser tests |
