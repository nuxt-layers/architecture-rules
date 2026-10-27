# Resources

> **In short:** every document outside this repository that its documents rely on, with why and where. This is the only place external links and pins appear ([ADR-0010](docs/decisions/ADR-0010-documentation-style.md)). Other documents name an entry here instead of linking out.

## Relied on

| Entry | What and why | Link |
|---|---|---|
| IAM suite architecture | The suite's architecture, state models and cross-capability processes, which ADR-0005 places in `iam-integration` | [architecture](https://github.com/nuxt-layers/iam-integration/blob/a51bffe7e4730192047c9fa47b0fcb5d28ee7fcd/docs/architecture.md), [states](https://github.com/nuxt-layers/iam-integration/blob/a51bffe7e4730192047c9fa47b0fcb5d28ee7fcd/docs/states.md), [processes](https://github.com/nuxt-layers/iam-integration/blob/a51bffe7e4730192047c9fa47b0fcb5d28ee7fcd/docs/processes/README.md) |
| Catalogue v0.4 | The full catalogue before it was reduced to existing capabilities: the definitions of every candidate capability in the [catalogue](docs/catalogue.md) | [v0.4](https://github.com/nuxt-layers/platform-architecture/blob/e0f03365fc16820e2043fa27f2f14bbb4f88c79f/docs/catalogue/nuxt-layers-capability-development-catalogue-v0.4.md) |

## For the record

Earlier versions of documents this repository has replaced or removed.

| Entry | What and why | Link |
|---|---|---|
| Catalogue v0.3 | The capability catalogue that ADR-0003 reconsidered: it placed Logging Service next | [v0.3](https://github.com/nuxt-layers/platform-architecture/blob/a8d5beef92806e01ba09b93f971b38d26c4ef6ba/docs/catalogue/nuxt-layers-capability-development-catalogue-v0.3.md) |
| Headless migration v0.1 | The superseded plan to move screens into kits; ADR-0009's amendment replaced it, apart from `ui`'s first release, which belongs to `ui`'s own plan | [v0.1](https://github.com/nuxt-layers/platform-architecture/blob/e0f03365fc16820e2043fa27f2f14bbb4f88c79f/docs/architecture/headless-migration-v01.md) |
| Architecture documents v0.1 | The two documents the [architecture overview](docs/architecture.md) replaced | [platform architecture](https://github.com/nuxt-layers/platform-architecture/blob/ffff2ba1c3078de543559a9e73d663739de54450/docs/architecture/platform-architecture-v01.md), [composition and dependency model](https://github.com/nuxt-layers/platform-architecture/blob/ffff2ba1c3078de543559a9e73d663739de54450/docs/architecture/composition-dependency-model-v01.md) |
| Identity and security documents v0.1 | The documents [Identity and Access](docs/identity.md) and [Security](docs/security.md) replaced | [identity and authorisation](https://github.com/nuxt-layers/platform-architecture/blob/b1bfb04012b49bc3776434426ac3e3d49d9e13ab/docs/identity/identity-authorization-architecture-v01.md), [group model](https://github.com/nuxt-layers/platform-architecture/blob/b1bfb04012b49bc3776434426ac3e3d49d9e13ab/docs/identity/group-model-definition-v01.md), [security architecture](https://github.com/nuxt-layers/platform-architecture/blob/b1bfb04012b49bc3776434426ac3e3d49d9e13ab/docs/security/security-architecture-v01.md), [data store security](https://github.com/nuxt-layers/platform-architecture/blob/b1bfb04012b49bc3776434426ac3e3d49d9e13ab/docs/standards/data-store-security-v01.md) |
| Documentation style v0.1 | The standard folded into [ADR-0010](docs/decisions/ADR-0010-documentation-style.md) | [documentation style](https://github.com/nuxt-layers/platform-architecture/blob/18379221786e16479d4ef9354a9a14ffa23a009f/docs/documentation-style.md) |
| Layer documents v0.1 | The six documents the [layer standard](docs/layer-standard.md) replaced | [layer architecture](https://github.com/nuxt-layers/platform-architecture/blob/1160d9806222cc00dad137f0b3275230221f92b1/docs/architecture/layer-architecture-v01.md), [interface standard](https://github.com/nuxt-layers/platform-architecture/blob/1160d9806222cc00dad137f0b3275230221f92b1/docs/contracts/layer-interface-standard-v01.md), [manifest](https://github.com/nuxt-layers/platform-architecture/blob/1160d9806222cc00dad137f0b3275230221f92b1/docs/contracts/capability-manifest-v01.md), [repository standard](https://github.com/nuxt-layers/platform-architecture/blob/1160d9806222cc00dad137f0b3275230221f92b1/docs/standards/layer-repository-standard-v01.md), [compatibility and versioning](https://github.com/nuxt-layers/platform-architecture/blob/1160d9806222cc00dad137f0b3275230221f92b1/docs/standards/compatibility-versioning-v01.md), [consumption workflow](https://github.com/nuxt-layers/platform-architecture/blob/1160d9806222cc00dad137f0b3275230221f92b1/docs/standards/layer-consumption-workflow-v01.md) |

## Repositories

The [catalogue](docs/catalogue.md) names each capability's repository; all are in the [`nuxt-layers`](https://github.com/nuxt-layers) organisation, at `github.com/nuxt-layers/<name>`.
