# Documentation
Last updated: 2026-03-21

This corpus is a fresh repository guide built from implementation, manifests, ADRs, tests, package READMEs, and support assets in the current checkout. When historical docs disagree with code, the code-backed architecture notes in this folder win and call the drift out explicitly.

## Start Here
| Page | Use it for |
|---|---|
| [`./architecture/README.md`](./architecture/README.md) | Master narrative: what Claude Flow / RuFlo V3.5 is, who it is for, how requests move, and how the repo is layered. |
| [`./architecture/repository-map.md`](./architecture/repository-map.md) | Coverage matrix for top-level areas, `v3/@claude-flow/*`, `v3/plugins/*`, legacy `v2/src`, and support assets. |
| [`./architecture/version-evolution-and-migration.md`](./architecture/version-evolution-and-migration.md) | How the repo moved from v2 to v3, what the root package now does, and what migration really means in this checkout. |
| [`../v3/docs/README.md`](../v3/docs/README.md) | Compatibility bridge for links coming from `v3/README.md`. |

## Architecture Set
| Topic | Primary page |
|---|---|
| Core runtime | [`./architecture/v3-core-runtime.md`](./architecture/v3-core-runtime.md) |
| CLI, MCP, hooks, operations, `ruflo/` | [`./architecture/v3-surfaces-and-operations.md`](./architecture/v3-surfaces-and-operations.md) |
| Security, claims, governance | [`./architecture/v3-security-governance.md`](./architecture/v3-security-governance.md) |
| Plugin SDK and shipped plugins | [`./architecture/v3-plugin-ecosystem.md`](./architecture/v3-plugin-ecosystem.md) |
| Legacy v2 line | [`./architecture/v2-legacy-architecture.md`](./architecture/v2-legacy-architecture.md) |
| Support assets and harnesses | [`./architecture/auxiliary-assets-and-harnesses.md`](./architecture/auxiliary-assets-and-harnesses.md) |

## What This Corpus Assumes
| Fact | What it means for readers |
|---|---|
| Root branding is now RuFlo V3.5 while package names still use `claude-flow` / `@claude-flow/*`. | Expect branding drift between README copy, manifests, and executable names. |
| The root `claude-flow` binary is a wrapper at [`../bin/cli.js`](../bin/cli.js). | The root package delegates into the v3 CLI implementation instead of being the implementation itself. |
| This checkout is source-first. | Several binaries and package entrypoints expect built `dist/` artifacts that are not present locally. |
| The active architecture is package-based. | Prefer the scoped v3 packages and the docs in `docs/architecture/` over older monolithic narratives. |

## Navigation by Question
| Question | Page |
|---|---|
| What is the product and who is it for? | [`./architecture/README.md`](./architecture/README.md) |
| Where does a request go after CLI or MCP entry? | [`./architecture/v3-surfaces-and-operations.md`](./architecture/v3-surfaces-and-operations.md) |
| What actually owns swarm, memory, integration, and learning? | [`./architecture/v3-core-runtime.md`](./architecture/v3-core-runtime.md) |
| How do plugins and packaged extensions fit in? | [`./architecture/v3-plugin-ecosystem.md`](./architecture/v3-plugin-ecosystem.md) |
| How does governance and execution safety work? | [`./architecture/v3-security-governance.md`](./architecture/v3-security-governance.md) |
| What still lives in v2, and how different is it? | [`./architecture/v2-legacy-architecture.md`](./architecture/v2-legacy-architecture.md), [`./architecture/version-evolution-and-migration.md`](./architecture/version-evolution-and-migration.md) |
| What do `.claude`, `.github`, hooks, scripts, and Docker tests do? | [`./architecture/auxiliary-assets-and-harnesses.md`](./architecture/auxiliary-assets-and-harnesses.md) |

## Coverage Notes
The repository map intentionally includes every top-level repo area, every scoped v3 package, every shipped `v3/plugins/*` package, the major `v2/src` subsystem families, and the support/operational surfaces requested for this documentation refresh.
