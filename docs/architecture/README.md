# Architecture Overview
Last updated: 2026-03-21

## Purpose
This is the master architecture narrative for the repository now branded as RuFlo V3.5 and still published through the `claude-flow` package line. It explains what the system is for, which layers are active, how a request moves through the runtime, and how to navigate the rest of the corpus.

## Status
| Scope | Status | Notes |
|---|---|---|
| Root package and workspace shell | `active` | The repository root publishes the branded entry surface and wraps the v3 CLI implementation. |
| V3 scoped packages | `active` | `v3/@claude-flow/*` is the live package-based runtime and API surface. |
| `ruflo/` branded wrapper and deployment stack | `active` | Adds the `ruflo` wrapper plus deployable MCP/UI assets. |
| V3 plugin ecosystem | `active` | `v3/plugins/*` ships domain, tooling, and reasoning extensions on top of the plugin SDK. |
| V2 runtime line | `legacy` | Still important for compatibility, migration context, and historical installs. |
| Support assets and harnesses | `supporting asset` | `.claude`, `.github`, hooks, scripts, and Docker harnesses keep the repo operable. |

## Covered Areas
| Area | Status | Why it matters |
|---|---|---|
| Product and target use cases | `active` | Claude Code / Codex augmentation, MCP serving, multi-agent orchestration, plugin-backed workflows, and branded `ruflo` deployments. |
| Architecture layers | `active` | Entry surfaces, control planes, coordination, memory/learning, providers, plugins, and operational assets. |
| Request lifecycle | `active` | Maps how CLI, MCP, hooks, swarm, memory, providers, security, and plugins fit together. |
| Repository navigation | `active` | Points to the focused architecture notes instead of forcing one giant file. |
| Legacy and migration context | `legacy` | Explains how the v2 line differs and what “migration” actually covers in the checked-in code. |

## Key Entry Points
| Surface | Key files | Role |
|---|---|---|
| Root publish shell | [`../../package.json`](../../package.json), [`../../bin/cli.js`](../../bin/cli.js), [`../../README.md`](../../README.md) | Root npm identity, branding, and wrapper entry into the v3 CLI. |
| V3 command and MCP ingress | [`../../v3/@claude-flow/cli/bin/cli.js`](../../v3/@claude-flow/cli/bin/cli.js), [`../../v3/@claude-flow/cli/src/index.ts`](../../v3/@claude-flow/cli/src/index.ts), [`../../v3/@claude-flow/mcp/src/server.ts`](../../v3/@claude-flow/mcp/src/server.ts) | CLI dispatch, MCP entry, and transport runtime. |
| Coordination and runtime core | [`../../v3/@claude-flow/swarm/src/unified-coordinator.ts`](../../v3/@claude-flow/swarm/src/unified-coordinator.ts), [`../../v3/@claude-flow/memory/src/hybrid-backend.ts`](../../v3/@claude-flow/memory/src/hybrid-backend.ts), [`../../v3/@claude-flow/integration/src/agentic-flow-bridge.ts`](../../v3/@claude-flow/integration/src/agentic-flow-bridge.ts), [`../../v3/@claude-flow/neural/src/sona-manager.ts`](../../v3/@claude-flow/neural/src/sona-manager.ts) | Canonical runtime path for orchestration, persistence, external delegation, and learning. |
| Governance and extension | [`../../v3/@claude-flow/security/src/index.ts`](../../v3/@claude-flow/security/src/index.ts), [`../../v3/@claude-flow/claims/src/index.ts`](../../v3/@claude-flow/claims/src/index.ts), [`../../v3/@claude-flow/plugins/src/index.ts`](../../v3/@claude-flow/plugins/src/index.ts) | Security boundary, ownership/claims, and plugin lifecycle. |
| Branded deployment wrapper | [`../../ruflo/bin/ruflo.js`](../../ruflo/bin/ruflo.js), [`../../ruflo/rvf.manifest.json`](../../ruflo/rvf.manifest.json) | RuFlo-branded CLI surface and deployable bridge/UI bundle. |
| Legacy line | [`../../v2/package.json`](../../v2/package.json), [`../../v2/src/cli/main.ts`](../../v2/src/cli/main.ts), [`../../v2/src/mcp/server.ts`](../../v2/src/mcp/server.ts) | Reference points for the legacy runtime and migration context. |

## How It Works
| Step | Runtime layer | What happens |
|---|---|---|
| 1 | Invocation | A user or client enters through the root `claude-flow` wrapper, the `ruflo` wrapper, a direct scoped CLI package, or an MCP client connection. |
| 2 | Surface selection | The CLI decides between command execution and MCP stdio mode; hooks and local runtime assets can also invoke the stack indirectly. |
| 3 | Validation and authority | Security, claims, and adjacent hook/browser controls validate input, ownership, and execution boundaries before privileged work continues. |
| 4 | Coordination | Swarm selects topology, consensus, worker routing, and agent delegation. |
| 5 | State and learning | Memory persists exact and semantic state, integration bridges to agentic-flow, and neural services record and distill patterns. |
| 6 | Externalization | Providers, browser services, and plugins supply model access, automation, and domain-specific behavior. |
| 7 | Operations | Testing, performance, deployment, CI, hooks, and support assets verify, package, and observe the system. |

The practical request path is therefore:

`user/client -> CLI or MCP -> validation and routing -> swarm/runtime core -> memory and learning -> providers or plugins -> result`

That path is not fully monolithic in this checkout. Some behavior still depends on wrappers, build outputs, optional peers, or support-layer configuration.

## Why It Is Designed This Way
| Design choice | Why it exists |
|---|---|
| Root wrapper plus scoped packages | Preserves one installable entry while letting the real runtime live in versioned packages. |
| Package-based v3 runtime | Keeps coordination, memory, security, hooks, providers, plugins, and deployment evolvable in smaller units. |
| Branded `ruflo` layer | Allows product branding and deployable bridge/UI surfaces to evolve without renaming every scoped package. |
| Legacy v2 kept in-tree | Supports migration, compatibility, and historical support cases while v3 matures. |
| Separate support assets | Claude runtime helpers, CI, installers, and container harnesses can change independently from the core runtime packages. |
| Explicit drift tracking | The repository contains conflicting historical narratives, so the docs call out where implementation wins. |

## Dependencies
| Dependency layer | Status | Notes |
|---|---|---|
| Node.js 20+ | `active` | Root and v3 manifests target current Node-based execution. |
| Build outputs under `dist/` | `supporting asset` | Several wrappers and package entrypoints expect compiled artifacts that are absent in this checkout. |
| AgentDB / SQLite / HNSW / SONA / RuVector | `active` | Memory, neural, and plugin paths rely on a mix of exact storage, vector search, and optional learning/native backends. |
| `agentic-flow` integration | `supporting asset` | V3 uses bridge-and-fallback patterns rather than treating the external core as always present. |
| Provider integrations | `supporting asset` | Higher-level surfaces call into model providers rather than owning LLM behavior directly. |
| CI, hooks, Docker, and scripts | `supporting asset` | Operational surfaces are required to verify and package the repo but are not the core runtime. |

## Operational/Test Notes
| Topic | Note |
|---|---|
| Source of truth | Prefer scoped package sources and manifests over historical marketing copy or stale root barrels. |
| Build sensitivity | The current checkout lacks many `dist/` artifacts, so wrapper binaries are not enough to prove runtime health without a build. |
| Request tracing | Use the focused docs for each layer instead of relying on one file to explain every package. |
| Legacy changes | Test v2 surfaces directly because its CLI, MCP, swarm, hive, and migration paths are intentionally split. |
| Security review | Pair runtime docs with the governance note because several trust boundaries sit outside the core swarm/memory path. |

## Known Drift
| Drift | Evidence | Why it matters |
|---|---|---|
| Brand and package identity diverge | Root README presents RuFlo V3.5 while the root package remains `claude-flow` and the scoped packages remain `@claude-flow/*`. | Users can read different names for the same system depending on entrypoint. |
| Root publish surface is a wrapper, not the implementation | [`../../package.json`](../../package.json) exposes [`../../bin/cli.js`](../../bin/cli.js), which in turn imports the v3 CLI path. | Top-level install behavior is one indirection away from the real code. |
| The known “root points directly at v3 CLI” fact is only partially true | The root wrapper delegates into `v3/@claude-flow/cli/bin/cli.js`, but the manifest itself points at the root wrapper first. | Packaging docs need to be precise about the extra compatibility layer. |
| Build artifacts are missing across active packages | Several docs in this corpus call out absent `dist/` output beneath active v3 packages and the `ruflo` wrapper. | Runtime invocations from source can fail even when the architecture is correct. |
| Historical counts and feature totals are unstable | Tool counts, agent counts, and capability numbers vary across README, ADRs, and support assets. | The corpus focuses on behavior and ownership rather than repeating unstable totals. |

## Related Docs
| Document | Why it matters |
|---|---|
| [`./repository-map.md`](./repository-map.md) | Full inventory of top-level areas, packages, plugins, and legacy subsystems. |
| [`./v3-core-runtime.md`](./v3-core-runtime.md) | Canonical runtime ownership for swarm, memory, integration, and learning. |
| [`./v3-surfaces-and-operations.md`](./v3-surfaces-and-operations.md) | Request entry surfaces, hooks, MCP, testing, performance, deployment, and `ruflo`. |
| [`./v3-security-governance.md`](./v3-security-governance.md) | Governance, claims, execution controls, and security drift. |
| [`./v3-plugin-ecosystem.md`](./v3-plugin-ecosystem.md) | Plugin SDK and shipped extension families. |
| [`./v2-legacy-architecture.md`](./v2-legacy-architecture.md) | Source-backed legacy runtime context. |
| [`./auxiliary-assets-and-harnesses.md`](./auxiliary-assets-and-harnesses.md) | Support assets, hooks, CI, scripts, and Docker harness details. |
| [`./version-evolution-and-migration.md`](./version-evolution-and-migration.md) | Version lineage and migration interpretation. |
