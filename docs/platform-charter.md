# Nuxt Layers — Platform Charter

> **Summary:** Build a platform that lets organisations create, configure and extend their own applications using reusable software components.

This charter sets the goals for the Platform Blueprint Review. It does not replace accepted ADRs, standards or public contracts.

## 1. Purpose

Nuxt Layers is an experiment in AI-assisted software development.

Its aim is to discover how far one experienced developer, working with AI, can build and maintain a large software platform that would traditionally require a team of specialists.

The project is intended to grow over several years and become commercially useful. Success depends on software quality, client value, manageable costs and the ability to keep developing it.

## 2. Vision

Organisations should be able to build applications from reusable features and adapt them as their needs change.

Authorised users should be able to configure their own:

- Organisational structures and responsibilities.
- Roles, permissions and access rules.
- Policies, approvals and business processes.
- Pages, forms, themes and templates.
- External application connections and automated tasks.

These changes should not require new application code wherever practical.

Clients should control their data, configurations and operations, subject to legal and security requirements. Applications should be straightforward to extend as new features become available.

## 3. Principles

| Principle | Rule |
|---|---|
| **Reuse before building** | Prefer established, actively maintained, permissively licensed open-source software. |
| **Client control** | Keep data portable, infrastructure replaceable and administration in the client's hands. |
| **Configure at runtime** | Let users define and change behaviour without redeployment wherever safe. |
| **Compose features** | Reuse functionality without requiring every feature to be a separate Nuxt Layer or repository. |
| **Keep ownership clear** | Each capability owns its data and rules. Shared tools must not duplicate that ownership. |
| **Build for change** | Start with a modular monolith. Separate services when there is a good reason. |
| **Protect existing work** | Refactor completed capabilities only when the benefits outweigh the costs. |
| **Challenge decisions** | Accepted ADRs govern current development but may be changed through formal review. |
| **Prove simplicity** | Building and extending ordinary applications must remain straightforward. |
| **Work critically with AI** | Use AI for design, implementation and independent review, with evidence and human approval. |

## 4. Review scope

The Platform Blueprint Review will examine:

| Area | Main question |
|---|---|
| Capabilities | Are responsibilities and software boundaries correctly defined? |
| Open source | Can existing permissively licensed projects meet our requirements? |
| Runtime configuration | How can users safely define their own rules, policies and processes? |
| Organisation and IAM | How should identities, organisational structures, permissions and governance cooperate? |
| Interfaces and templates | How can users design pages, forms, workflows and reusable templates? |
| Integrations | How can applications connect to external services and marketplaces? |
| Capability catalogue | How can applications and development tools discover available functionality? |
| Data and deployment | How can clients retain control while the platform grows? |
| Development method | How can AI-assisted development remain efficient, reliable and affordable? |

The review must consider both completed and proposed capabilities. It must identify unnecessary complexity, duplication, missing functionality and opportunities to reuse existing software.

## 5. Working method

The review will follow the existing architectural authority and development rules.

For each area:

1. Establish what users and applications need.
2. Examine accepted ADRs and relevant public contracts.
3. Investigate suitable open-source solutions.
4. Compare alternatives, benefits, costs and risks.
5. Recommend changes only where justified.
6. Record decisions and measurable acceptance criteria.

Existing implementations will not be changed during the blueprint review. New architectural proposals must be approved before becoming governing rules.

The review will use existing token-saving and repository-access restrictions. Detailed investigations must remain focused and independently reviewable.

## 6. Success criteria

The review succeeds when we have:

- A clear blueprint for the platform and its parts.
- A consistent classification of completed and proposed capabilities.
- Defined approaches for runtime policies, templates, workflows and user interfaces.
- Clear rules for external integrations and client data ownership.
- Identified reusable open-source foundations.
- A practical method for discovering, composing and testing capabilities.
- A justified list of changes to existing architecture.
- Reference applications demonstrating that the design works for different organisations.

The blueprint must guide development without becoming a second source of architectural authority.

## Governing authority

`nuxt-layers/architecture-rules` remains authoritative.

This charter defines the project's aims and the questions the review must answer. It does not replace accepted ADRs, standards or capability contracts.

Changes to governing architecture require the normal ADR process.
