# Contributing

> **In short:** focused pull requests, checked and accepted by the project owner; AI does the engineering under human direction, to the same standards as any contribution.

Changes come as focused branches and pull requests, each passing `python3 scripts/check_markdown.py`, and are accepted by the project owner. This repository's documents follow [ADR-0010](docs/decisions/ADR-0010-documentation-style.md). How the work is done is described below.

## AI-driven development

### Purpose and scope

Nuxt Layers is an experimental, AI-driven, specification-led open-source software engineering initiative. This document explains its development methodology, accountability model and quality expectations.

It applies to repositories in the `nuxt-layers` organisation and to the independent consumer and integration repositories that adopt it.

This document describes **development process**, not new architectural authority. The established documentation hierarchy, accepted ADRs, repository standards and security policies remain authoritative.

### Operating model

AI assistants perform the principal engineering activities under human direction and governance. Work may include requirements analysis, architectural proposals, implementation, refactoring, tests, security assessment, documentation, integration verification and maintenance.

The human project owner retains final authority for requirements, scope, architectural acceptance, risk decisions, repository governance, merges and releases. AI output is a proposal or engineering contribution, not an autonomous approval.

The workflow is AI-driven, **not human-free**. The use of multiple assistants does not itself constitute independent verification.

### Engineering principles

1. **Specification-led:** consult current authoritative documentation and accepted decisions before making changes. Do not infer normative architecture solely from implementation.
2. **Bounded and replaceable:** preserve capability boundaries, explicit contracts, separation of concerns and one-way dependencies.
3. **Evidence-based:** substantiate claims using repository state, test results, review findings and reproducible evidence. Distinguish verified facts from assumptions.
4. **Secure by design:** apply the project's current security and privacy standards; treat AI-generated code as untrusted until reviewed and verified.
5. **Accessible and maintainable:** follow the platform's accessibility, internationalisation, documentation and maintainability requirements.
6. **Traceable change:** use focused branches and pull requests, explain rationale, preserve history and respect protected branches and release policies.
7. **Equal standards:** apply the same review, tests, policies and acceptance criteria to human-authored and AI-generated contributions.

### Typical development lifecycle

1. **Establish context:** verify the live repository baseline, relevant specifications, existing tests and dependencies.
2. **Define the change:** identify the problem, boundaries, acceptance criteria, risks and affected consumers.
3. **Design:** propose the smallest coherent change consistent with the governing architecture; record material architectural decisions through the established ADR process.
4. **Implement:** make focused, reviewable changes without unrelated modifications.
5. **Verify:** run applicable linting, type checking, unit, integration, end-to-end, security and accessibility checks; report gaps and failures explicitly.
6. **Review and accept:** inspect the diff and evidence, resolve failures, and obtain the project owner's acceptance through the repository's protected-branch workflow.
7. **Release and maintain:** follow the repository's versioning, dependency pinning, provenance and release policies; update consumer guidance when contracts change.

The exact checks and release process depend on the repository's existing policies. A green CI run is necessary where required, but does not by itself prove correctness or security.

### Responsibility and limitations

AI assistance can accelerate engineering and identify defects, but may introduce errors, hallucinate dependencies, misunderstand requirements or overlook security risks. Human acceptance and automated quality controls mitigate these risks without eliminating them.

Do not claim a security audit, conformance assessment, successful test, independent review or production readiness unless supporting evidence exists. Document unresolved limitations and known risks.

Never disclose credentials or sensitive information to AI tools or commit them to source control. Follow the repository's security reporting and secret-management procedures.

### Transparency and repository notices

Participating repositories should display a concise **AI-Driven Development** notice near the top of their root `README.md` and link to this repository's `CONTRIBUTING.md`. The notice should accurately describe AI's engineering role, human decision authority and equal quality standards.

The independent platform test harness may use the same notice while identifying itself as a consumer verification project rather than an organisation-owned capability.

This methodology does not change the licence, contributor obligations, intellectual-property requirements, security policy or normative architectural hierarchy of any repository.

### Objective

The initiative explores whether disciplined, specification-led AI development can deliver secure, maintainable, standards-compliant, production-quality open-source software. Those qualities are objectives to be demonstrated through evidence, not guarantees conferred by the development method.
