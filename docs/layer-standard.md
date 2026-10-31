# Layer Standard

> **In short:** what a layer is, how its repository is laid out and documented, what it offers and keeps private, what its manifest says, and how an application consumes it. It replaces the layer architecture, layer interface standard, capability manifest, layer repository standard, layer consumption workflow and compatibility and versioning standard. The decisions it rests on are ADR-0001, ADR-0002, ADR-0008, ADR-0009 and ADR-0012.

**Status:** Normative

## 1. What a layer is

A layer is a Nuxt Layer implementing one bounded capability. It gets its own repository only when separate ownership, reuse, testing or security gives clear value; a component, composable, entity or endpoint alone never does. Prefer cohesive ownership to small packages: UI primitives belong in `ui`, sign-in in Authentication, users, groups and memberships in Identity.

| Class | Is | Examples |
|---|---|---|
| Foundation | Reusable cross-cutting capabilities | Identity, Authentication, Authorisation, Theme Manager |
| Platform | Reusable platform functions | Publisher, media, notifications |
| Domain | Coherent business or end-user capabilities | clubs, bookings |
| Presentation | Screens and components over capabilities' contracts, owning no data | `ui`, `content-editor`, `gallery` |
| Library | Stateless packages with no storage, ports or screens | `rich-text` |

Foundation, Platform and Domain layers are capabilities: a headless core with an optional presentation part ([ADR-0009](decisions/ADR-0009-headless-capability-layers.md)).

## 2. Public surface and contracts

A layer's public surface is what it deliberately offers; everything else is private and may change without a public contract change. It may offer:

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

Each independently developed layer documents:

- its purpose and bounded responsibility, public contract, provided and required capabilities, configuration, compatibility and testing;
- security and privacy where they apply, with a threat model and control register for security-critical capabilities;
- for a capability, its *Headless surface*: what any screen drawing it must still do (ADR-0009);
- `CONTRACT.md`, its one-page offer to hosts ([ADR-0012](decisions/ADR-0012-trivial-composition.md) §6);
- `CHANGELOG.md`, `RESOURCES.md` and, while it has planned work, `PLAN.md`, all at the root; Resources names the ADRs and other documents it relies on ([ADR-0010](decisions/ADR-0010-documentation-style.md)).

Default branches are protected; material changes come through pull requests, and architecture-changing work names the governing ADR.

## 4. The manifest

Each independently developed layer ships `capability.json`, so tooling and hosts can reason about what it provides and needs. It agrees with the human-readable contract and package metadata, and never overrides them.

```json
{
  "schemaVersion": "1",
  "name": "@nuxt-layers/example",
  "version": "0.0.0",
  "classification": "foundation",
  "provides": [{ "capability": "ExampleService", "contractVersion": "1" }],
  "requires": [{ "capability": "IdentityService", "contractVersion": "^1", "optional": false }]
}
```

The sample `version` is development metadata, not a published release or consumer pin. It names the manifest schema version, the package, its development metadata version, its class (`foundation`, `platform`, `domain`, `presentation` or `library`), what it provides with contract versions, and what it requires with compatible versions and whether each is optional. Later versions may add configuration schemas, events, runtime needs, security metadata and compatibility evidence.

## 5. Testing

Each layer has contract tests for its public surface, unit tests for its rules, integration tests for its adapters, compatibility tests for the capabilities it requires, security tests fitting its threat surface, and `examples/minimal-host` proving its composition budget (ADR-0012). A host also tests the composed system.

## 6. Consuming a layer

During active development, consumers integrate the latest accepted `master` of each dependency (ADR-0008). Local workspace paths may accelerate development, but acceptance testing must resolve the current upstream `master`.

| Situation | Method |
|---|---|
| Belongs only to the application | A local `layers/` layer |
| Developed alongside others locally | A workspace or local path, followed by upstream integration |
| Its own repository | A Git-backed dependency following the latest accepted `master` |
| A future published release | Governed by a separate release decision when releases resume |

Do not copy another layer's source into an application or use a Git submodule merely to avoid dependency integration. Direct remote `extends` remains exceptional and must be justified and tested.

Whenever a provider's `master` advances, affected consumers refresh their dependency resolution and run contract, integration, security and acceptance checks. A committed lockfile records the resolved inputs of that check; it must not freeze integration to an older development revision. Failed checks require a fix in the responsible provider or consumer before the consumer's integration is accepted.

## 7. Versioning

Formal package releases, release tags and compatibility obligations to superseded development revisions are suspended until the project owner approves a release lifecycle (ADR-0008). Contract changes remain explicit, documented and tested, even when breaking changes are expected during development. Contract-version fields describe interface compatibility, not repository dependency pinning; they must be updated when the interface changes.

Capability and contract metadata may retain schema versions required by tooling; such fields do not imply a formal package-release lifecycle. Consumer compatibility is established against current upstream `master`, not a historic tag.
