# Repository Map
Last updated: 2026-03-21

## Purpose
This document is the coverage matrix for the repository. It maps the top-level workspace areas, every scoped v3 package, every shipped `v3/plugins/*` package, the major legacy `v2/src` subsystem families, and the support assets that keep the repo operable.

## Status
| Scope | Status | Notes |
|---|---|---|
| Top-level repo map | `active` | Tracks the current workspace shape rather than historical architecture descriptions. |
| V3 scoped packages | `active` | These are the live implementation units. |
| V3 shipped plugins | `active` | Shipped extension packages under `v3/plugins/*`. |
| V2 subsystem map | `legacy` | Important for compatibility, support, and migration context. |
| Support and operational assets | `supporting asset` | Hooks, CI, scripts, and runtime helpers that influence behavior. |

## Covered Areas
| Coverage set | Status | Included here |
|---|---|---|
| Top-level repo areas | `active` | Every current top-level directory in the workspace root. |
| `v3/@claude-flow/*` packages | `active` | All scoped packages found in the active v3 tree. |
| `v3/plugins/*` packages | `active` | Every shipped plugin package in the plugins tree. |
| Major `v2/src` subsystem families | `legacy` | Legacy runtime, tooling, operations, and experimental families. |
| Support assets named in the refresh request | `supporting asset` | `.claude`, `.claude-plugin`, `plugin/`, `.github`, `.githooks`, `scripts/`, `tests/docker-regression`, and `docs/ruvector-postgres` coverage. |

## Key Entry Points
| Area | Key files |
|---|---|
| Root publish shell | [`../../package.json`](../../package.json), [`../../bin/cli.js`](../../bin/cli.js), [`../../README.md`](../../README.md) |
| V3 workspace | [`../../v3/package.json`](../../v3/package.json), [`../../v3/pnpm-workspace.yaml`](../../v3/pnpm-workspace.yaml), [`../../v3/README.md`](../../v3/README.md) |
| V2 workspace | [`../../v2/package.json`](../../v2/package.json), [`../../v2/README.md`](../../v2/README.md) |
| Branded wrapper | [`../../ruflo/package.json`](../../ruflo/package.json), [`../../ruflo/bin/ruflo.js`](../../ruflo/bin/ruflo.js) |
| Live Claude runtime assets | [`../../.claude/settings.json`](../../.claude/settings.json), [`../../.claude/mcp.json`](../../.claude/mcp.json) |
| Legacy/newer plugin surfaces | [`../../.claude-plugin/plugin.json`](../../.claude-plugin/plugin.json), [`../../plugin/hooks/hooks.json`](../../plugin/hooks/hooks.json) |

## How It Works
The repository is layered around one active implementation line and several compatibility/support lines:

1. The root package is the publish shell and brand surface.
2. `v3/` contains the active scoped package runtime, plus the shipped plugin packages that extend it.
3. `ruflo/` adds the branded wrapper and deployable bridge/UI stack.
4. `v2/` preserves the older runtime line for compatibility and migration context.
5. Support assets such as `.claude`, `.github`, scripts, and Docker harnesses make the system operable, testable, and installable.

### Top-level workspace areas
| Path | Status | Role |
|---|---|---|
| `.agents/` | `supporting asset` | Local skills, scripts, and agent coordination assets used by contributors and automation. |
| `.claude/` | `active` | Live Claude Code runtime helpers, statusline, local MCP wiring, and command assets. |
| `.claude-plugin/` | `legacy` | Older plugin-distribution surface and installer metadata. |
| `.codex/` | `supporting asset` | Codex-local configuration and companion automation assets. |
| `.git/` | `supporting asset` | Repository metadata, history, and branch state. |
| `.githooks/` | `active` | Commit-time redaction/security hook. |
| `.github/` | `active` | CI, verification, integration, rollback, badges, and issue templates. |
| `agents/` | `supporting asset` | Small checked-in YAML archetype set for live agents. |
| `bin/` | `active` | Root wrapper binaries. |
| `docs/` | `active` | Fresh architecture corpus plus prompt material. |
| `plugin/` | `experimental` | Newer wrapper/hook plugin surface. |
| `ruflo/` | `active` | Branded wrapper package and deployable bridge/UI stack. |
| `scripts/` | `supporting asset` | Install, verify, and cleanup helpers. |
| `tests/` | `supporting asset` | Regression and integration test assets, including Docker harnesses. |
| `v2/` | `legacy` | Legacy runtime line and migration/support context. |
| `v3/` | `active` | Current scoped packages, plugins, ADRs, and runtime implementation. |

### Scoped V3 packages
| Package | Status | Role |
|---|---|---|
| `aidefence` | `supporting asset` | AI-manipulation detection and learned mitigation helpers. |
| `agents` | `supporting asset` | Config-only agent definitions in the v3 tree. |
| `browser` | `active` | Browser automation and MCP/browser security surfaces. |
| `claims` | `active` | Ownership, handoff, rebalance, and contest logic. |
| `cli` | `active` | Main command-line surface and MCP-mode wrapper. |
| `codex` | `supporting asset` | Codex adapter, generators, migrations, and dual-mode helpers. |
| `deployment` | `supporting asset` | Release, publish, and version management. |
| `embeddings` | `supporting asset` | Embedding services, hyperbolic helpers, and persistent cache. |
| `guidance` | `active` | Governance and long-horizon control-plane package. |
| `hooks` | `active` | Hook registry, executor, daemons, statusline, and hook MCP tools. |
| `integration` | `active` | Bridge into `agentic-flow` and adjacent acceleration paths. |
| `mcp` | `active` | Standalone MCP runtime with transports and tool registry. |
| `memory` | `active` | SQLite, AgentDB, hybrid backend, HNSW, and persistence helpers. |
| `neural` | `active` | SONA, ReasoningBank, modes, and learning orchestration. |
| `performance` | `supporting asset` | Benchmark and attention-validation tooling. |
| `plugins` | `active` | Plugin SDK, registry, dependency graph, and collections. |
| `providers` | `supporting asset` | Multi-LLM provider routing and failover. |
| `security` | `active` | Validation, path controls, safe execution, signing, and credential helpers. |
| `shared` | `active` | Common types, events, resilience, plugin contracts, and MCP helpers. |
| `swarm` | `active` | Canonical coordination runtime, topology, consensus, and workers. |
| `testing` | `supporting asset` | Fixtures, mocks, regression helpers, and compatibility tests. |

### Shipped V3 plugins
| Plugin | Status | Functional group |
|---|---|---|
| `agentic-qe` | `active` | Dev tooling and quality engineering |
| `code-intelligence` | `active` | Dev tooling and analysis |
| `cognitive-kernel` | `experimental` | Reasoning and optimization |
| `financial-risk` | `active` | Vertical and regulatory |
| `gastown-bridge` | `experimental` | Coordination and external bridge |
| `healthcare-clinical` | `active` | Vertical and regulatory |
| `hyperbolic-reasoning` | `experimental` | Reasoning and optimization |
| `legal-contracts` | `active` | Vertical and regulatory |
| `neural-coordination` | `experimental` | Coordination and orchestration |
| `perf-optimizer` | `active` | Dev tooling and optimization |
| `prime-radiant` | `experimental` | Reasoning, interpretability, and verification |
| `quantum-optimizer` | `experimental` | Reasoning and optimization |
| `ruvector-upstream` | `supporting asset` | Shared substrate and bridge layer |
| `teammate-plugin` | `active` | Coordination and integration |
| `test-intelligence` | `active` | Dev tooling and test optimization |

### Major `v2/src` subsystem families
| Family | Status | Directories covered |
|---|---|---|
| Entry and user surfaces | `legacy` | `cli`, `api`, `terminal`, `templates` |
| Core execution model | `legacy` | `core`, `agents`, `task`, `execution`, `resources`, `monitoring`, `permissions`, `integration`, `workflows` |
| Coordination systems | `legacy` | `swarm`, `hive-mind`, `communication` |
| Protocol and intelligence layers | `legacy` | `mcp`, `hooks`, `neural`, `reasoningbank`, `modes`, `providers`, `services`, `sdk` |
| Enterprise and governance | `legacy` | `enterprise`, `verification`, `migration` |
| Data, plumbing, and infrastructure | `legacy` | `db`, `config`, `constants`, `types`, `utils`, `patches` |
| Test and fixture surfaces | `supporting asset` | `tests`, `__tests__` |
| Compatibility and experimental families | `historical/unverified` | `automation`, `maestro`, `mle-star`, `consciousness-symphony` |

### Support and operational surfaces
| Path | Status | Notes |
|---|---|---|
| `.claude` | `active` | Live local runtime helpers and MCP/hook assets. |
| `.claude-plugin` | `legacy` | Older plugin-distribution flow. |
| `plugin/` | `experimental` | Newer hook surface with partial manifest coverage. |
| `.github` | `active` | CI and verification workflows. |
| `.githooks` | `active` | Commit-time redaction gate. |
| `scripts/` | `supporting asset` | Install/verify/cleanup automation. |
| `tests/docker-regression/` | `supporting asset` | Containerized regression harness. |
| `docs/ruvector-postgres/` | `historical/unverified` | No directory exists in this checkout. |

## Why It Is Designed This Way
| Design choice | Benefit |
|---|---|
| Root shell + scoped runtime | Separates publishing and branding from the actual implementation units. |
| V3 package split | Keeps high-change subsystems isolated and easier to document. |
| Plugin subtree | Allows domain/tooling extensions to evolve on top of one shared SDK. |
| In-tree v2 line | Preserves support and migration context without merging it into v3. |
| Separate support assets | Makes local runtime, CI, installers, and test harnesses visible as first-class surfaces. |

## Dependencies
| Layer | Dependency shape |
|---|---|
| Root | Depends on wrapper entrypoints and packaged v3 CLI files. |
| V3 runtime | Depends on Node 20+, package builds, optional AgentDB/SONA/RuVector/`agentic-flow` integrations, and internal package imports. |
| V3 plugins | Depend on the plugin SDK plus a mix of `zod`, optional native/WASM peers, and substrate bridges. |
| V2 runtime | Depends on legacy MCP, CLI, ReasoningBank, provider, and migration plumbing. |
| Support assets | Depend on shell tools, Docker, Git, CI runners, and local Claude/Codex tooling. |

## Operational/Test Notes
| Topic | Note |
|---|---|
| Inventory source | This map is based on the current tree and focused implementation notes, not on historical architecture descriptions. |
| Build state | Package maps do not imply that the local checkout is runnable without a build. |
| Legacy context | Treat v2 family names as support/migration boundaries, not as current recommended extension points. |
| Support assets | Operational surfaces can materially affect runtime behavior, especially around hooks, MCP, and CI. |

## Known Drift
| Drift | Why it matters |
|---|---|
| The root workspace contains both current and legacy packaging surfaces. | Readers need explicit boundaries between live runtime, wrappers, and historical installer flows. |
| `docs/ruvector-postgres/` is part of the requested repo story but is absent in this checkout. | Any docs claiming that tree exists would be misleading. |
| `v3/pnpm-workspace.yaml` covers scoped packages but not the shipped plugin packages. | Workspace membership and shipped package scope do not line up cleanly. |
| Several active packages expect built `dist/` output that is not present locally. | Repo shape and runnable state are not the same thing in this checkout. |

## Related Docs
| Document | Why it matters |
|---|---|
| [`./README.md`](./README.md) | Master architecture narrative. |
| [`./v3-core-runtime.md`](./v3-core-runtime.md) | Runtime ownership behind the package map. |
| [`./v3-plugin-ecosystem.md`](./v3-plugin-ecosystem.md) | Detailed plugin grouping and entrypoints. |
| [`./v2-legacy-architecture.md`](./v2-legacy-architecture.md) | Detailed v2 legacy behavior and drift. |
| [`./auxiliary-assets-and-harnesses.md`](./auxiliary-assets-and-harnesses.md) | Support assets and operational harness details. |
