# Layer Standard

> **In short:** what a layer is, how its repository is laid out and documented, what it offers and keeps private, what its manifest says, and how an application consumes it. It replaces the layer architecture, layer interface standard, capability manifest, layer repository standard, layer consumption workflow and compatibility and versioning standard. The decisions it rests on are ADR-0001, ADR-0002, ADR-0008, ADR-0009 and ADR-0012.

**Status:** Normative

## 1. What a layer is

A layer is a Nuxt Layer implementing one bounded capability. It gets its own repository only when separate ownership, versioning, reuse, testing, release or security gives clear value; a component, composable, entity or endpoint alone never does. Prefer cohesive ownership to small packages: UI primitives belong in `ui`, sign-in in Authentication, users, groups and memberships in Identity.

| Class | Is | Examples |
|---|---|---|
| Foundation | Reusable cross-cutting capabilities | Identity, Authentication, Authorisation, Theme Manager |
| Platform | Reusable platform functions | Publisher, media, notifications |
| Domain | Coherent business or end-user capabilities | clubs, bookings |
| Presentation | Screens and components over capabilities' contracts, owning no data | `ui`, `content-editor`, `gallery` |
| Library | Stateless packages with no storage, ports or screens | `rich-text` |

Foundation, Platform and Domain layers are capabilities: a headless core with an optional presentation part ([ADR-0009](decisions/ADR-0009-headless-capability-layers.md)).

## 2. Public surface and contracts

A layer's public surface is what it deliberately offers; everything else is private and may change without a breaking release. It may offer:

| Contract | Is |
|---|---|
| Service | Callable server functions, one task each |
| Data, command, query | Stable cross-boundary values, requested changes and reads |
| Event | Facts published after something happened, through the layer's outbox |
| Configuration | Validated settings for the host |
| Error | Documented failure codes and `messageKey`s, never wording |
| HTTP API | Endpoints the host may switch off |
| Port | What the layer asks the host to supply, in ADR-0012's shared shapes |
| Typed client | Data and actions for screens |
| Presentation part | Optional pages, components and a message catalogue, separable from the core (ADR-0009) |

Rules:

- Contracts use the domain's language and never expose a replaceable provider's SDK types, unless the architecture makes that provider part of the contract.
- Public exports come from a deliberate entry point, such as `/contracts`, or a documented export map.
- Consumers never import internal paths, private server code, persistence adapters, database schemas or undocumented files.
- Input crossing a trust boundary is validated at run time; TypeScript types are not validation.
- Removing a public member, narrowing accepted input, changing documented meaning or changing a public shape incompatibly is a breaking change. An addition is not automatically safe: consider its effect on behaviour and types.
- Whether a repository is public or private does not change what its contract is.

## 3. The repository

```text
/
├── app/          screens, if the layer has a presentation part
├── server/
├── shared/
├── tests/
├── docs/
├── examples/minimal-host/
├── CHANGELOG.md
├── CONTRACT.md
├── PLAN.md        while there is planned work
├── README.md
├── RESOURCES.md
├── capability.json
├── nuxt.config.ts
└── package.json
```

Only the directories the capability needs exist. Packages use the `@nuxt-layers/` scope; repository names are short capability names, and repository and package boundaries align. Every imported dependency is declared.

Each independently versioned layer documents:

- its purpose and bounded responsibility, public contract, provided and required capabilities, configuration, compatibility and testing;
- security and privacy where they apply, with a threat model and control register for security-critical capabilities;
- for a capability, its *Headless surface*: what any screen drawing it must still do (ADR-0009);
- `CONTRACT.md`, its one-page offer to hosts ([ADR-0012](decisions/ADR-0012-trivial-composition.md) §6);
- `CHANGELOG.md`, `RESOURCES.md` and, while it has planned work, `PLAN.md`, all at the root; Resources names the ADRs and other documents it relies on ([ADR-0010](decisions/ADR-0010-documentation-style.md)).

Default branches are protected; material changes come through pull requests, and architecture-changing work names the governing ADR.

## 4. The manifest

Each independently versioned layer ships `capability.json`, so tooling and hosts can reason about what it provides and needs. It agrees with the human-readable contract and package metadata, and never overrides them.

```json
{
  "schemaVersion": "1",
  "name": "@nuxt-layers/example",
  "version": "1.0.0",
  "classification": "foundation",
  "provides": [{ "capability": "ExampleService", "contractVersion": "1" }],
  "requires": [{ "capability": "IdentityService", "contractVersion": "^1", "optional": false }]
}
```

It names the manifest schema version, the package, its version, its class (`foundation`, `platform`, `domain`, `presentation` or `library`), what it provides with contract versions, and what it requires with compatible versions and whether each is optional. Later versions may add configuration schemas, events, runtime needs, security metadata and compatibility evidence.

## 5. Testing

Each layer has contract tests for its public surface, unit tests for its rules, integration tests for its adapters, compatibility tests for the capabilities it requires, security tests fitting its threat surface, and `examples/minimal-host` proving its composition budget (ADR-0012). A host also tests the composed system.

## 6. Consuming a layer

| Situation | Method |
|---|---|
| Belongs only to the application | A local `layers/` layer |
| Developed alongside others locally | A workspace or local path, restored before release |
| Its own repository | A Git-backed dependency, pinned to a release tag's commit (or an immutable commit before its first release) |
| A stable published package | The versioned package |

Never copy a layer's source into an application, follow a default branch, or use a Git submodule for a new layer. Direct remote `extends` is exceptional: say why, pin a revision and test it in CI.

Adding or updating a layer is an explicit change:

1. Read its release notes and contract changes.
2. Set the version or pinned commit.
3. Install and commit the lockfile, which records Git dependencies as `codeload.github.com` tarballs, never `git@github.com` ([ADR-0008](decisions/ADR-0008-independent-layer-delivery.md)).
4. Name the package in Nuxt `extends` and supply what its `CONTRACT.md` asks.
5. Run the application's type, test, security and build checks.

The pull request shows the version or commit, the lockfile change, and any compatibility, configuration, contract, security, privacy or migration impact.

When a Git-backed layer gains a stable lifecycle, it is published to the approved registry. Its consumers switch to the package version, update the lockfile, run composition tests, and remove Git-specific configuration. The repository stays the source.

## 7. Versioning

- Released layers use semantic versioning from 1.0.0 ([ADR-0008](decisions/ADR-0008-independent-layer-delivery.md)). From then, a breaking change to the public contract or its documented meaning is a major release; before 1.0.0, breaking changes are still recorded.
- From 1.0.0, a removal is deprecated in a minor release, with migration guidance, before the major release that removes it.
- Contract versions may move independently of package versions; the manifest names those provided and required.
- Changing a private provider without changing the contract needs no major release, unless provider behaviour is part of the documented contract.
- A layer tests against the dependency ranges it supports; a host tests the exact set it deploys.
