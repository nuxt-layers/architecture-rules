# ADR-0011 — Shared Rich Text and Configurable Copy

**Status:** Accepted  
**Date:** 2026-10-21  
**Amended:** 2026-10-21: plain text is not rich text; `rich-text` is a library, not a layer; the editor is used only where formatting is allowed  
**Amended and accepted:** 2026-10-22: `ui` draws safe trees; wording keys placed; Profile and Configuration adopt it in their headless releases  
**Amended:** 2026-10-26: in line with the amended ADR-0009 (layers keep their screens) and content-editor's design

> **In short:** every piece of copy is data. Plain text is an ordinary string in an ordinary form control. Formatted text is written in one shared format, checked by the layer that stores it, and edited, where a screen wants it, through one editor that offers exactly what the field allows.

## Why

Copy appears everywhere: published articles, a person's bio, a site's notices, a button's label. If each layer chose its own format and safety rules, the same text would behave differently in each place, and one weak spot would put every reader at risk. Copy written into code cannot be changed without a release.

Most copy needs no formatting at all. A name or a label is a string, and a standard input edits it perfectly well. Only text that carries formatting, such as emphasis, links, lists or tables, needs a shared format and an editor.

## The decision

### 1. Plain text stays plain

A label, name, title or setting is a **plain string** with a maximum length, validated by the owning layer's schema and edited with standard form controls (`<input>`, `<textarea>`). It never uses `rich-text` or `content-editor`.

### 2. One format for formatted text, in a small library

The platform has one rich-text format: a safe subset of Markdown. It lives in **`nuxt-layers/rich-text`**, a stateless library, not a layer: no pages, routes, storage or ports. It holds:

- the format's rules: which constructs and links are allowed;
- the parser, which turns text into a **safe tree** that can never carry markup;
- the checks a layer runs before saving;
- the profiles below.

### 3. Profiles, from a caption to a document

Every formatted field declares a **profile** and a maximum length.

| Profile | Allows | Typical use |
|---|---|---|
| **Inline** | Bold, italic, links | A tagline, a caption, a short notice |
| **Block** | Paragraphs, lists, headings, quotes | A bio, a description, a summary |
| **Document** | Everything, including tables, images and links between documents | An article or a page |

### 4. The layer that stores it enforces it

The capability that owns a field checks every save against its profile, using `rich-text`. Anything beyond the profile is refused, whichever screen sent it. Readers receive only the safe tree.

`ui` draws safe trees, in the browser and on the server. Output that is not a page, such as an email or a full-content feed, is drawn from the same tree by a `ui` function when its first consumer needs it; no capability writes HTML.

### 5. The editor where formatting needs it

**`nuxt-layers/content-editor`** is the platform's editing component for formatted fields, built on `ui` and `rich-text`. It reads a field's profile and offers only those tools. The field editor edits text and hands it back, knowing nothing about the capability that stores it; content-editor's own screens for Publisher's documents work through the functions the host passes, never by importing Publisher.

A screen uses it only where a field's formatting needs more than a standard control can give. A plain `<textarea>` remains a valid choice for any formatted field: safety comes from the storing layer's checks, never from the editor.

### 6. No hard-coded copy

Text a person reads is data, never code.

| Copy | Lives in |
|---|---|
| Published content | The publisher |
| A person's own words | Profile |
| A site's or organisation's text | Configuration |
| Interface labels and messages | The message catalogue of whichever presentation part draws them (a layer's own, `ui` or a kit), keyed by the codes and `messageKey`s capabilities publish; the application overrides any of them ([ADR-0009](ADR-0009-headless-capability-layers.md)) |

### 7. Capabilities may depend on stateless libraries

A capability may depend on a `nuxt-layers` library that holds no state, no ports and no screens, such as `rich-text`, as it depends on `zod`. Capabilities still never depend on one another ([ADR-0001](ADR-0001-capability-repositories-and-composition-shells.md)).

## Adopting it

| Layer | Adopts it |
|---|---|
| `rich-text` | Created from the publisher's existing dialect, with the three profiles added |
| `publisher` | Uses `rich-text` for formatted fields before its 1.0 release; its own dialect is removed |
| `content-editor` | Built on `ui` and `rich-text` as a presentation layer ([ADR-0009](ADR-0009-headless-capability-layers.md)) |
| Profile and Configuration | Move fields longer than a label to `rich-text` in their next major release, so their contracts break once; plain fields stay as they are |
| Others | Use `rich-text` for formatted fields in their next major release |

## What follows

| Gains | Costs |
|---|---|
| Formatted text behaves and is protected the same everywhere | Every capability with formatted fields depends on one library, so its releases must be careful |
| Plain fields stay simple: a string and a standard control | Each field must be classed as plain or formatted, and a profile chosen for the formatted ones |
| Copy changes without a release | Interface wording needs translation resources the application can edit |
| One editor to learn, build and test, used only where it earns its place | `content-editor` must serve a caption and an article equally well |

## Guardrails

- `rich-text` never renders HTML; it returns safe trees only.
- A plain field never accepts formatting; turning it into a formatted field is a contract change for its layer.
- A change that loosens a profile, or widens what the format allows, needs a new ADR.
- A stateless library holds no storage, ports, screens or configuration of its own.
