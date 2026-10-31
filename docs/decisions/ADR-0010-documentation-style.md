# ADR-0010 — Documentation Style

**Status:** Accepted  
**Date:** 2026-10-21  
**Accepted:** 2026-10-22  
**Amended:** 2026-10-25: documents that stay true without upkeep; one file for external links, one for plans  
**Amended:** 2026-10-26: `CONTRACT.md` listed with the other files; `RESOURCES.md` and `PLAN.md` at the repository root; the documentation style standard folded in

> **In short:** every document should be quick to read and hard to misread, and should stay true without being edited. Brevity comes first. Verified external reference pins and changing status stay in their designated files; live development dependencies follow ADR-0008.

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
| `RESOURCES.md` | Every accessible external architectural document relied on, one row each, identifying the verified revision reviewed (tag or commit). Live software dependencies follow ADR-0008. This is the only place external architectural links and verified revisions appear; software dependency resolution follows ADR-0008 | Anything else |
| `PLAN.md` | Every plan and its status: what is done, in progress and next, replacing roadmaps and registers. Present only while the repository has planned work: the architecture repository has none, as plans belong to each capability and to the applications driving the order of work | Decisions' reasoning, which stays in the documents it belongs to |
| `CHANGELOG.md` | Major changes, newest first, with dates; formal release versions resume when the release lifecycle is approved | Plans, status, reasoning or links |
| `CONTRACT.md` | A capability's one-page offer to hosts and other layers ([ADR-0012](ADR-0012-trivial-composition.md) §6), with its development status and public interface; a suite describes the members it composes without release pins during development | Any other repository's versions, plans or links; what it needs from elsewhere is named through `RESOURCES.md` |

Every other document refers to an external document by its entry in `RESOURCES.md`, with a short note on why, never by its own link. During development, `CONTRACT.md` describes current composition without release pins (ADR-0008).

## Adopting it

New documents follow it from the start. Existing documents, and each repository's files and Markdown check, are brought into line by the first piece of work that next changes that repository, never by a chat of its own.

## What follows

| Gains | Costs |
|---|---|
| Faster reading and review | Rewriting older documents over time |
| Fewer misreadings by people and AI | Editing for brevity takes effort |
| A verified reference update touches one file, not many | Readers follow one more step to reach an external document |
