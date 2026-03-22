# Version Evolution and Migration
Last updated: 2026-03-21

## Purpose
This document explains how the repository evolved from the legacy v2 line to the current v3 package architecture and the RuFlo V3.5 brand surface. It also clarifies what migration means in this checkout, because the code, manifests, and READMEs use that word in different ways.

## Status
| Line | Status | Notes |
|---|---|---|
| Root `claude-flow` package | `active` | Current publish shell and compatibility wrapper. |
| RuFlo brand surface | `active` | Product and deployment branding in the root README and `ruflo/` package. |
| V3 scoped packages | `active` | Current implementation line. |
| V2 runtime | `legacy` | Maintained for compatibility and migration context. |
| V2 migration tooling | `legacy` | Real migration tooling exists, but it migrates prompts/config/assets rather than the runtime architecture itself. |

## Covered Areas
| Area | Status | Included here |
|---|---|---|
| Naming evolution | `active` | `claude-flow` package identity versus RuFlo product branding. |
| Runtime evolution | `active` | Monolithic/split legacy runtime versus scoped v3 packages. |
| Wrapper evolution | `active` | Root `bin/cli.js` and `ruflo/bin/ruflo.js` delegation behavior. |
| Migration scope | `legacy` | What the v2 migration CLI actually does. |
| Operator guidance | `supporting asset` | How to read and work with the mixed-version repo safely. |

## Key Entry Points
| Surface | Key files | Why they matter |
|---|---|---|
| Root shell | [`../../package.json`](../../package.json), [`../../bin/cli.js`](../../bin/cli.js) | Shows how the root package now delegates into the v3 CLI implementation. |
| Product branding | [`../../README.md`](../../README.md), [`../../ruflo/package.json`](../../ruflo/package.json), [`../../ruflo/bin/ruflo.js`](../../ruflo/bin/ruflo.js) | Establishes the RuFlo V3.5 presentation layer. |
| Active runtime line | [`../../v3/package.json`](../../v3/package.json), [`../../v3/@claude-flow/cli/bin/cli.js`](../../v3/@claude-flow/cli/bin/cli.js), [`../../v3/src/index.ts`](../../v3/src/index.ts) | Shows where the live package-based runtime now sits. |
| Legacy runtime line | [`../../v2/package.json`](../../v2/package.json), [`../../v2/src/cli/main.ts`](../../v2/src/cli/main.ts), [`../../v2/src/mcp/server.ts`](../../v2/src/mcp/server.ts) | Shows how the older runtime is still structured. |
| Legacy migration tooling | [`../../v2/src/migration/index.ts`](../../v2/src/migration/index.ts), [`../../v2/src/migration/README.md`](../../v2/src/migration/README.md) | Clarifies the real migration contract for v2 users. |

## How It Works
| Phase | What changed |
|---|---|
| v2 legacy line | The runtime was broad and heterogeneous: multiple entrypoints, multiple coordination systems, multiple memory surfaces, and an MCP layer that sat beside the CLI instead of clearly underneath it. |
| v3 transition | Implementation moved into scoped packages under `v3/@claude-flow/*`, with dedicated packages for CLI, MCP, hooks, providers, memory, swarm, plugins, security, and support tooling. |
| RuFlo brand layer | The root README and `ruflo/` package added a product/deployment surface without renaming the published package family. |
| Current root behavior | Installing `claude-flow` still lands on the root package, but the executable first enters [`../../bin/cli.js`](../../bin/cli.js), which then imports the v3 CLI wrapper. |
| Migration reality | The checked-in v2 migration CLI updates project prompts, configs, and backups. It does not automatically convert consumers from the v2 runtime architecture to the v3 package architecture. |

### Practical migration reading
| If you are... | Read this as... |
|---|---|
| A root-package user | The root install is a compatibility wrapper over the v3 CLI, not a separate runtime line. |
| A v2 maintainer | v2 is still present for support and migration context, but it should be documented as legacy. |
| A package integrator | The canonical future-facing imports live under `v3/@claude-flow/*`, not under the root package or `v3/index.ts`. |
| A deployment/UI operator | `ruflo/` is the branded bridge/UI surface and should be treated separately from the core scoped packages. |

## Why It Is Designed This Way
| Design choice | Why it exists |
|---|---|
| Keep the root package name | Preserves install continuity and ecosystem discoverability. |
| Add RuFlo branding on top | Allows broader product positioning without forcing a hard rename of every package and binary overnight. |
| Move implementation into scoped packages | Reduces cross-subsystem entanglement and gives each runtime surface a clearer owner. |
| Keep v2 in-tree | Supports older installs, migration tooling, and historical debugging without flattening everything into v3. |
| Use compatibility wrappers | Makes it possible to evolve internals while keeping user-facing entrypoints familiar. |

## Dependencies
| Surface | Dependency or contract |
|---|---|
| Root wrapper | Depends on `v3/@claude-flow/cli/bin/cli.js` being present and usable. |
| V3 runtime | Depends on package-local builds, internal scoped package imports, and optional external acceleration backends. |
| RuFlo wrapper | Depends on the v3 CLI plus the bridge/UI/data stack declared in `ruflo/`. |
| V2 migration CLI | Depends on legacy project files, manifest rules, and backup helpers rather than on v3 package conversion logic. |

## Operational/Test Notes
| Topic | Note |
|---|---|
| Migration language | Do not assume “migration” means v2-to-v3 runtime conversion unless the code path explicitly says so. |
| Root installs | Test the root and `ruflo` wrappers after builds, because both rely on built output from the v3 CLI. |
| Consumer guidance | Point new code at scoped v3 packages and treat v2 as compatibility context. |
| Documentation maintenance | When root branding, wrapper behavior, or migration tooling changes, this page should be updated alongside the affected manifests and READMEs. |

## Known Drift
| Drift | Evidence | Why it matters |
|---|---|---|
| Product name and package name differ | Root README says RuFlo V3.5, but root npm identity is still `claude-flow`. | Branding and installation instructions can sound like they refer to different products. |
| The root binary does not point directly at the v3 CLI in `package.json` | [`../../package.json`](../../package.json) exposes [`../../bin/cli.js`](../../bin/cli.js), and only that wrapper imports the v3 CLI path. | Packaging facts need to distinguish between manifest wiring and runtime delegation. |
| V2 “migration” is narrower than the repo-wide migration story | The v2 migration CLI focuses on prompts/config/assets and backup flows. | Users can overestimate how much automation exists for moving architecture and integrations forward. |
| Workspace membership and shipped scope differ in v3 | The scoped package workspace does not cleanly include the shipped `v3/plugins/*` packages. | Build and packaging expectations need to be documented separately from source layout. |
| Current wrappers are build-dependent | The active wrappers expect `dist/` artifacts that are absent in this checkout. | Version evolution is correct in architecture, but not immediately runnable from raw source. |

## Related Docs
| Document | Why it matters |
|---|---|
| [`./README.md`](./README.md) | Master narrative for the current architecture. |
| [`./repository-map.md`](./repository-map.md) | Coverage matrix for the mixed-version repository shape. |
| [`./v2-legacy-architecture.md`](./v2-legacy-architecture.md) | Detailed legacy behavior and drift. |
| [`./v3-core-runtime.md`](./v3-core-runtime.md) | Current runtime ownership. |
| [`./v3-surfaces-and-operations.md`](./v3-surfaces-and-operations.md) | Entry surfaces and operational layers around the active runtime. |
| [`../../v3/README.md`](../../v3/README.md) | Historical V3 overview and package-level orientation. |
