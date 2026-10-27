# ADR-0010 — Documentation Style

**Status:** Accepted  
**Date:** 2026-10-21  
**Accepted:** 2026-10-22  
**Amended:** 2026-10-25: documents that stay true without upkeep; one file for external links, one for plans  
**Amended:** 2026-10-26: `CONTRACT.md` listed with the other files; `RESOURCES.md` and `PLAN.md` at the repository root; the documentation style standard folded in

> **In short:** every document should be quick to read and hard to misread, and should stay true without being edited. Brevity comes first. Versions, commits and status are written in one place per repository, never spread through its documents.

## Why

Our documents are accurate but often long and dense. Readers, human and AI alike, spend time finding the decision inside the detail.

## The decision

Every document in the ecosystem follows these rules:

| Rule | In practice |
|---|---|
| **Summary first** | Open with a one-line *In short* |
| **Brevity** | Cut every word that does not change the meaning |
| **Plain language** | Everyday words; a term or piece of jargon is explained where it first appears |
| **Say it once** | Every rule a reader needs is stated once, in one place; elsewhere, link to it |
| **Principles first** | Examples only where they help |
| **Short sections** | One idea each, under a clear heading |
| **Tables for comparisons** | Options, owners, gains and costs side by side |
| **Lists for steps and sets** | Numbered for order, bullets otherwise |
| **Bold for key terms** | Sparingly, so the eye can scan |
| **British spelling** | Throughout |

An ADR reads in this order: *In short*, **Why**, **The decision**, **Adopting it**, **What follows** (gains and costs, as a table), and **Guardrails** where needed.

### Documents that stay true

Keeping versions, pins and status up to date across many files costs time and tokens and does nothing for the product. So each repository keeps changing facts in a few files at its root, each with one job; every other document lives under `docs/`:

| File | Holds | Never holds |
|---|---|---|
| `README.md`, `CODE_OF_CONDUCT.md`, `CONTRIBUTING.md`, `SECURITY.md` | What the repository is and how to work with it, in general terms. Links only to the repository's own documents | Version numbers, release names, pinned commits, status, plans or links outside the repository |
| `RESOURCES.md` | Every external document the repository relies on, one table row each: what it is, why it is needed, and its pin (tag or commit). Apart from `CONTRACT.md`'s own pins, this is the only place external links and versions appear | Anything else |
| `PLAN.md` | Every plan and its status: what is done, in progress and next, replacing roadmaps and registers. Present only while the repository has planned work: the architecture repository has none, as plans belong to each capability and to the applications driving the order of work | Decisions' reasoning, which stays in the documents it belongs to |
| `CHANGELOG.md` | Each release and major change, newest first: its version, date and a short description of what changed | Plans, status, reasoning or links |
| `CONTRACT.md` | A capability's one-page offer to hosts and other layers ([ADR-0012](ADR-0012-trivial-composition.md) §6), with its own release status and pinned version; a suite's contract also pins the members it composes, because they are part of what it offers | Any other repository's versions, plans or links; what it needs from elsewhere is named through `RESOURCES.md` |

Every other document refers to an external document by its entry in `RESOURCES.md`, with a short note on why, never by its own link. `CONTRACT.md`'s own pins are the only versions written outside `RESOURCES.md`.

## Adopting it

New documents follow it from the start. Existing documents, and each repository's files and Markdown check, are brought into line by the first piece of work that next changes that repository, never by a chat of its own.

## What follows

| Gains | Costs |
|---|---|
| Faster reading and review | Rewriting older documents over time |
| Fewer misreadings by people and AI | Editing for brevity takes effort |
| A release or a new pin touches one file, not many | Readers follow one more step to reach an external document |
