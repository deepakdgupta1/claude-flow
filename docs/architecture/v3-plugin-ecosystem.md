# V3 Plugin Ecosystem
Last updated: 2026-03-21

## Purpose
This document captures the plugin SDK/lifecycle core in `v3/@claude-flow/plugins` and the shipped plugin packages in `v3/plugins/*`, based on implementation and manifest evidence in this workspace. It groups plugins by function, not by directory listing, so the ecosystem can be read as a set of domain layers built on one shared plugin runtime.

## Status
| Scope | Status | Notes |
|---|---|---|
| `@claude-flow/plugins` | `active` | SDK, workers, hooks, providers, registry, collections, and RuVector integration are implemented and exported from the package. |
| Vertical/regulatory plugins | `active` | `healthcare-clinical`, `financial-risk`, and `legal-contracts` are shipped TS packages with domain-specific bridges and tools. |
| Dev tooling plugins | `active` | `agentic-qe`, `code-intelligence`, `test-intelligence`, and `perf-optimizer` are shipped as tool-focused packages. |
| Coordination/integration plugins | `experimental` | `neural-coordination`, `teammate-plugin`, and `gastown-bridge` add orchestration or bridge behavior with more integration surface area. |
| Reasoning/optimization plugins | `experimental` | `cognitive-kernel`, `quantum-optimizer`, `hyperbolic-reasoning`, and `prime-radiant` push deeper reasoning/math paths. |
| Shared substrate | `supporting asset` | `ruvector-upstream` centralizes reusable vector, attention, graph, hyperbolic, learning, and SONA bridges. |

## Covered Areas
### SDK and lifecycle core
| Package | Status | Role | Representative entry points |
|---|---|---|---|
| `@claude-flow/plugins` | `active` | Plugin lifecycle, builders, registries, workers, hooks, providers, dependency graph, collections, and RuVector bridge wiring. | [src/index.ts](../../v3/@claude-flow/plugins/src/index.ts), [src/core/plugin-interface.ts](../../v3/@claude-flow/plugins/src/core/plugin-interface.ts), [src/core/base-plugin.ts](../../v3/@claude-flow/plugins/src/core/base-plugin.ts), [src/sdk/index.ts](../../v3/@claude-flow/plugins/src/sdk/index.ts), [src/workers/index.ts](../../v3/@claude-flow/plugins/src/workers/index.ts), [src/hooks/index.ts](../../v3/@claude-flow/plugins/src/hooks/index.ts), [src/providers/index.ts](../../v3/@claude-flow/plugins/src/providers/index.ts), [src/registry/plugin-registry.ts](../../v3/@claude-flow/plugins/src/registry/plugin-registry.ts), [src/registry/enhanced-plugin-registry.ts](../../v3/@claude-flow/plugins/src/registry/enhanced-plugin-registry.ts), [src/registry/dependency-graph.ts](../../v3/@claude-flow/plugins/src/registry/dependency-graph.ts), [src/collections/official/index.ts](../../v3/@claude-flow/plugins/src/collections/official/index.ts), [src/integrations/ruvector/index.ts](../../v3/@claude-flow/plugins/src/integrations/ruvector/index.ts) |

### Vertical and regulatory plugins
| Package | Status | Role | Representative entry points |
|---|---|---|---|
| `@claude-flow/plugin-healthcare-clinical` | `active` | Clinical decision support, patient similarity, and safety-oriented retrieval. | [package.json](../../v3/plugins/healthcare-clinical/package.json), [src/index.ts](../../v3/plugins/healthcare-clinical/src/index.ts), [src/mcp-tools.ts](../../v3/plugins/healthcare-clinical/src/mcp-tools.ts), [src/bridges/index.ts](../../v3/plugins/healthcare-clinical/src/bridges/index.ts) |
| `@claude-flow/plugin-financial-risk` | `active` | Risk analysis, sparse optimization, and finance-facing scoring. | [package.json](../../v3/plugins/financial-risk/package.json), [src/index.ts](../../v3/plugins/financial-risk/src/index.ts), [src/mcp-tools.ts](../../v3/plugins/financial-risk/src/mcp-tools.ts), [src/bridges/index.ts](../../v3/plugins/financial-risk/src/bridges/index.ts) |
| `@claude-flow/plugin-legal-contracts` | `active` | Contract reasoning, review, and attention/DAG-assisted analysis. | [package.json](../../v3/plugins/legal-contracts/package.json), [src/index.ts](../../v3/plugins/legal-contracts/src/index.ts), [src/mcp-tools.ts](../../v3/plugins/legal-contracts/src/mcp-tools.ts), [src/bridges/index.ts](../../v3/plugins/legal-contracts/src/bridges/index.ts) |

### Dev tooling plugins
| Package | Status | Role | Representative entry points |
|---|---|---|---|
| `@claude-flow/plugin-agentic-qe` | `active` | Quality engineering, test generation, coverage, chaos, and security automation. | [package.json](../../v3/plugins/agentic-qe/package.json), [src/index.ts](../../v3/plugins/agentic-qe/src/index.ts), [src/plugin.ts](../../v3/plugins/agentic-qe/src/plugin.ts), [src/tools/index.ts](../../v3/plugins/agentic-qe/src/tools/index.ts), [src/bridges/index.ts](../../v3/plugins/agentic-qe/src/bridges/index.ts), `agents/*.yaml`, `plugin.yaml` |
| `@claude-flow/plugin-code-intelligence` | `active` | Semantic code search, architecture analysis, and refactoring impact. | [package.json](../../v3/plugins/code-intelligence/package.json), [src/index.ts](../../v3/plugins/code-intelligence/src/index.ts), [src/mcp-tools.ts](../../v3/plugins/code-intelligence/src/mcp-tools.ts), [src/bridges/index.ts](../../v3/plugins/code-intelligence/src/bridges/index.ts) |
| `@claude-flow/plugin-test-intelligence` | `active` | Predictive test selection, flaky detection, and coverage optimization. | [package.json](../../v3/plugins/test-intelligence/package.json), [src/index.ts](../../v3/plugins/test-intelligence/src/index.ts), [src/mcp-tools.ts](../../v3/plugins/test-intelligence/src/mcp-tools.ts), [src/bridges/index.ts](../../v3/plugins/test-intelligence/src/bridges/index.ts) |
| `@claude-flow/plugin-perf-optimizer` | `active` | Bottleneck detection, memory analysis, and performance tuning. | [package.json](../../v3/plugins/perf-optimizer/package.json), [src/index.ts](../../v3/plugins/perf-optimizer/src/index.ts), [src/mcp-tools.ts](../../v3/plugins/perf-optimizer/src/mcp-tools.ts), [src/bridges/index.ts](../../v3/plugins/perf-optimizer/src/bridges/index.ts) |

### Coordination and integration plugins
| Package | Status | Role | Representative entry points |
|---|---|---|---|
| `@claude-flow/plugin-neural-coordination` | `experimental` | Coordination, topology optimization, and neural routing. | [package.json](../../v3/plugins/neural-coordination/package.json), [src/index.ts](../../v3/plugins/neural-coordination/src/index.ts), [src/mcp-tools.ts](../../v3/plugins/neural-coordination/src/mcp-tools.ts), [src/bridges/index.ts](../../v3/plugins/neural-coordination/src/bridges/index.ts) |
| `@claude-flow/teammate-plugin` | `active` | Native teammate bridge for multi-agent coordination and Claude Code integration. | [package.json](../../v3/plugins/teammate-plugin/package.json), [src/index.ts](../../v3/plugins/teammate-plugin/src/index.ts), [src/mcp-tools.ts](../../v3/plugins/teammate-plugin/src/mcp-tools.ts), [src/semantic-router.ts](../../v3/plugins/teammate-plugin/src/semantic-router.ts), [src/topology-optimizer.ts](../../v3/plugins/teammate-plugin/src/topology-optimizer.ts) |
| `@claude-flow/plugin-gastown-bridge` | `experimental` | WASM-accelerated bridge for formulas, convoy tracking, cache, and memory plumbing. | [package.json](../../v3/plugins/gastown-bridge/package.json), [src/index.ts](../../v3/plugins/gastown-bridge/src/index.ts), [src/mcp-tools.ts](../../v3/plugins/gastown-bridge/src/mcp-tools.ts), [src/wasm-loader.ts](../../v3/plugins/gastown-bridge/src/wasm-loader.ts), `scripts/build-wasm.sh`, `wasm/*` |

### Reasoning and optimization plugins
| Package | Status | Role | Representative entry points |
|---|---|---|---|
| `@claude-flow/plugin-cognitive-kernel` | `experimental` | Cognitive/SONA-style reasoning bridge. | [package.json](../../v3/plugins/cognitive-kernel/package.json), [src/index.ts](../../v3/plugins/cognitive-kernel/src/index.ts), [src/mcp-tools.ts](../../v3/plugins/cognitive-kernel/src/mcp-tools.ts), [src/bridges/index.ts](../../v3/plugins/cognitive-kernel/src/bridges/index.ts) |
| `@claude-flow/plugin-quantum-optimizer` | `experimental` | Optimization, DAG/exotic bridging, and combinatorial search. | [package.json](../../v3/plugins/quantum-optimizer/package.json), [src/index.ts](../../v3/plugins/quantum-optimizer/src/index.ts), [src/mcp-tools.ts](../../v3/plugins/quantum-optimizer/src/mcp-tools.ts), [src/bridges/index.ts](../../v3/plugins/quantum-optimizer/src/bridges/index.ts) |
| `@claude-flow/plugin-hyperbolic-reasoning` | `experimental` | Hyperbolic reasoning with GNN-assisted inference. | [package.json](../../v3/plugins/hyperbolic-reasoning/package.json), [src/index.ts](../../v3/plugins/hyperbolic-reasoning/src/index.ts), [src/mcp-tools.ts](../../v3/plugins/hyperbolic-reasoning/src/mcp-tools.ts), [src/bridges/index.ts](../../v3/plugins/hyperbolic-reasoning/src/bridges/index.ts) |
| `@claude-flow/plugin-prime-radiant` | `experimental` | Interpretability, coherence checks, consensus verification, and math engines. | [package.json](../../v3/plugins/prime-radiant/package.json), [src/index.ts](../../v3/plugins/prime-radiant/src/index.ts), [src/plugin.ts](../../v3/plugins/prime-radiant/src/plugin.ts), [src/engines/index.ts](../../v3/plugins/prime-radiant/src/engines/index.ts), [src/tools/index.ts](../../v3/plugins/prime-radiant/src/tools/index.ts), [src/wasm-bridge.ts](../../v3/plugins/prime-radiant/src/wasm-bridge.ts), `plugin.yaml` |

### Shared substrate
| Package | Status | Role | Representative entry points |
|---|---|---|---|
| `@claude-flow/ruvector-upstream` | `supporting asset` | Shared vector substrate for attention, HNSW, GNN, hyperbolic, learning, exotic, cognitive, and SONA bridges. | [package.json](../../v3/plugins/ruvector-upstream/package.json), [src/index.ts](../../v3/plugins/ruvector-upstream/src/index.ts), [src/registry.ts](../../v3/plugins/ruvector-upstream/src/registry.ts), [src/bridges/hnsw.ts](../../v3/plugins/ruvector-upstream/src/bridges/hnsw.ts), [src/bridges/attention.ts](../../v3/plugins/ruvector-upstream/src/bridges/attention.ts), [src/bridges/gnn.ts](../../v3/plugins/ruvector-upstream/src/bridges/gnn.ts), [src/bridges/hyperbolic.ts](../../v3/plugins/ruvector-upstream/src/bridges/hyperbolic.ts), [src/bridges/learning.ts](../../v3/plugins/ruvector-upstream/src/bridges/learning.ts), [src/bridges/exotic.ts](../../v3/plugins/ruvector-upstream/src/bridges/exotic.ts), [src/bridges/cognitive.ts](../../v3/plugins/ruvector-upstream/src/bridges/cognitive.ts), [src/bridges/sona.ts](../../v3/plugins/ruvector-upstream/src/bridges/sona.ts) |

## Key Entry Points
| Layer | Entry points |
|---|---|
| SDK and lifecycle core | [src/index.ts](../../v3/@claude-flow/plugins/src/index.ts), [src/core/plugin-interface.ts](../../v3/@claude-flow/plugins/src/core/plugin-interface.ts), [src/core/base-plugin.ts](../../v3/@claude-flow/plugins/src/core/base-plugin.ts), [src/sdk/index.ts](../../v3/@claude-flow/plugins/src/sdk/index.ts), [src/workers/index.ts](../../v3/@claude-flow/plugins/src/workers/index.ts), [src/hooks/index.ts](../../v3/@claude-flow/plugins/src/hooks/index.ts), [src/providers/index.ts](../../v3/@claude-flow/plugins/src/providers/index.ts), [src/registry/plugin-registry.ts](../../v3/@claude-flow/plugins/src/registry/plugin-registry.ts), [src/registry/enhanced-plugin-registry.ts](../../v3/@claude-flow/plugins/src/registry/enhanced-plugin-registry.ts), [src/registry/dependency-graph.ts](../../v3/@claude-flow/plugins/src/registry/dependency-graph.ts) |
| Collections and bridge wiring | [src/collections/official/index.ts](../../v3/@claude-flow/plugins/src/collections/official/index.ts), [src/integrations/ruvector/index.ts](../../v3/@claude-flow/plugins/src/integrations/ruvector/index.ts), [src/integrations/ruvector/ruvector-bridge.ts](../../v3/@claude-flow/plugins/src/integrations/ruvector/ruvector-bridge.ts) |
| Shared substrate | [src/index.ts](../../v3/plugins/ruvector-upstream/src/index.ts), [src/registry.ts](../../v3/plugins/ruvector-upstream/src/registry.ts), [src/bridges/hnsw.ts](../../v3/plugins/ruvector-upstream/src/bridges/hnsw.ts), [src/bridges/attention.ts](../../v3/plugins/ruvector-upstream/src/bridges/attention.ts), [src/bridges/gnn.ts](../../v3/plugins/ruvector-upstream/src/bridges/gnn.ts), [src/bridges/hyperbolic.ts](../../v3/plugins/ruvector-upstream/src/bridges/hyperbolic.ts), [src/bridges/learning.ts](../../v3/plugins/ruvector-upstream/src/bridges/learning.ts), [src/bridges/exotic.ts](../../v3/plugins/ruvector-upstream/src/bridges/exotic.ts), [src/bridges/cognitive.ts](../../v3/plugins/ruvector-upstream/src/bridges/cognitive.ts), [src/bridges/sona.ts](../../v3/plugins/ruvector-upstream/src/bridges/sona.ts) |

## How It Works
`@claude-flow/plugins` is the control layer. `PluginBuilder`, `BasePlugin`, the registry modules, and the dependency graph define how plugins are created, registered, versioned, and resolved. `workers`, `hooks`, and `providers` extend that lifecycle into execution, event handling, and model routing.

The shipped packages are thin domain facades over that core. Most expose `src/index.ts` plus one domain surface such as `src/mcp-tools.ts`, `src/bridges/*`, `src/tools/*`, or `src/wasm-bridge.ts`. That keeps the package contract small while allowing each plugin to specialize around QA, clinical/regulatory workflows, orchestration, or reasoning.

`ruvector-upstream` acts as the reusable substrate. Instead of reimplementing vector search, attention, graph, hyperbolic, or learning primitives in every plugin, the ecosystem routes those capabilities through a shared bridge layer.

## Why It Is Designed This Way
| Design choice | Benefit |
|---|---|
| Thin SDK plus plugin packages | The lifecycle contract stays centralized, while domain logic can evolve independently. |
| Registry and dependency graph separation | Plugin discovery, compatibility checks, and registration can be reasoned about separately. |
| Shared RuVector substrate | Vector and attention primitives are not duplicated across every vertical or tooling package. |
| Domain-specific package facades | Each shipped plugin can expose only the tools, bridges, or engines it actually needs. |
| Optional peers and native/WASM edges | Packages can degrade gracefully when advanced substrates are unavailable. |

## Dependencies
| Package group | Notable direct dependencies | Notes |
|---|---|---|
| `@claude-flow/plugins` | `events`; optional peers `@claude-flow/hooks`, `@claude-flow/memory`, `@ruvector/wasm`, `@ruvector/learning-wasm` | The core stays lightweight and treats the larger runtime surfaces as optional. |
| `agentic-qe` | `zod`; optional peers `@claude-flow/plugins`, `@claude-flow/memory`, `@claude-flow/security`, `@claude-flow/embeddings`, `@claude-flow/browser`, `@ruvector/*` | This package is the heaviest shipped tooling surface and assumes a broad QA stack. |
| `code-intelligence`, `test-intelligence`, `perf-optimizer`, `healthcare-clinical` | `zod`; optional `@claude-flow/ruvector-upstream` or related RuVector WASM peers | These packages are thin adapters around the shared substrate. |
| `financial-risk`, `legal-contracts`, `neural-coordination`, `cognitive-kernel`, `hyperbolic-reasoning` | `zod` plus package-local bridges | The manifests are small and push most behavior into source modules. |
| `gastown-bridge` | `@iarna/toml`, `uuid`, `zod`, plus Rust/WASM crates under `wasm/` | This is the clearest multi-language build in the set. |
| `prime-radiant` | `zod`; optional `prime-radiant-advanced-wasm`; peers `@claude-flow/memory`, `@claude-flow/security`, `@claude-flow/coordination` | Advanced math paths are optional rather than hard-required. |
| `teammate-plugin` | `eventemitter3`, `@ruvnet/bmssp`, optional `@anthropic-ai/claude-code` peer | This package bridges Claude Code teammate behavior directly. |
| `ruvector-upstream` | `zod`; optional `@ruvector/*-wasm` peers | Shared substrate for downstream plugins and experiments. |

## Operational/Test Notes
All targeted packages are source-only in this workspace: every checked `dist/` directory under `v3/@claude-flow/plugins` and `v3/plugins/*` is absent, even though the package manifests point at built outputs.

| Area | What to verify |
|---|---|
| `@claude-flow/plugins` | Run the package tests around registry, enhanced registry, dependency graph, collections, SDK, security, and RuVector bridges before changing lifecycle code. |
| JS plugin packages | Run `typecheck` and `test` in each package, then confirm the source entrypoints match the exported subpaths in `package.json`. |
| `agentic-qe` | Check `plugin.yaml`, `agents/*.yaml`, and the quality/security tool suites together so manifest and source stay aligned. |
| `gastown-bridge` | Run `build:wasm` before `build`, then `test:wasm` when Rust/WASM code changes. |
| `prime-radiant` | Verify `typecheck`, `build`, and the engine/tool tests, including the optional WASM bridge path. |
| `ruvector-upstream` | Exercise the bridge exports against the expected optional RuVector peers or mocks, not just `src/index.ts`. |

## Known Drift
- Every target package in this slice points at `dist/*` outputs, but this workspace does not currently contain any of those `dist/` directories. Source and manifest shape are present; built artifacts are not.
- `v3/@claude-flow/plugins/package.json` uses `./dist/*.d.js` in its `exports.types` entries, while the top-level `types` field points at `dist/index.d.ts`. That looks like a packaging typo rather than an intentional alternate extension.
- `@claude-flow/ruvector-upstream` and `@claude-flow/teammate-plugin` use bare `dist/...` paths in `main` and `types`, while most other plugin packages use `./dist/...`. The inconsistency is small but real.
- `gastown-bridge` spans TypeScript, Cargo, and WASM build paths. A package can look structurally complete in source while still being runtime-incomplete until the Rust/WASM side is built.
- `prime-radiant` relies on an optional `prime-radiant-advanced-wasm` dependency for advanced features, so a minimal install may not exercise the full math surface.

## Related Docs
- [v3/@claude-flow/plugins/README.md](../../v3/@claude-flow/plugins/README.md)
- [v3/plugins/agentic-qe/README.md](../../v3/plugins/agentic-qe/README.md)
- [v3/plugins/code-intelligence/README.md](../../v3/plugins/code-intelligence/README.md)
- [v3/plugins/gastown-bridge/README.md](../../v3/plugins/gastown-bridge/README.md)
- [v3/plugins/prime-radiant/README.md](../../v3/plugins/prime-radiant/README.md)
- [v3/plugins/ruvector-upstream/README.md](../../v3/plugins/ruvector-upstream/README.md)
- [v3/plugins/teammate-plugin/README.md](../../v3/plugins/teammate-plugin/README.md)
