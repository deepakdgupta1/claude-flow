# Plugin System

## Overview

The Plugin System implements a **microkernel architecture** (per ADR-004) that keeps the Claude Flow core minimal while enabling unlimited extensibility through plugins. Plugins can add MCP tools, hooks, workers, providers, and custom extension points. A decentralized IPFS-based marketplace enables community plugin distribution.

**Key Locations**:

| Path | Description |
|------|-------------|
| `v3/src/infrastructure/plugins/` | Core plugin infrastructure (PluginManager, BasePlugin, ExtensionPoint) |
| `v3/@claude-flow/plugins/` | Plugin SDK, builder API, marketplace integration |
| `v3/plugins/` | 15 domain-specific plugin implementations |

---

## Core Plugin Infrastructure

### PluginManager

The `PluginManager` class (per ADR-004) manages the full plugin lifecycle — loading, unloading, reloading, dependency resolution, and extension point invocation.

**Constructor**: `PluginManager(options?: PluginManagerOptions)`
- `eventBus`: Optional EventEmitter for lifecycle events
- `coreVersion`: Version string for compatibility checks (default: `'3.0.0'`)

**Key Methods**:

| Method | Description |
|--------|-------------|
| `loadPlugin(plugin, config?)` | Validates dependencies, checks version compatibility, initializes plugin, registers extension points |
| `unloadPlugin(pluginId)` | Shuts down plugin, removes all its extension points |
| `reloadPlugin(pluginId, plugin)` | Unloads then reloads a plugin (hot-reload) |
| `listPlugins()` | Returns all loaded plugins |
| `getPluginMetadata(pluginId)` | Returns id, name, version, description, author, homepage |
| `invokeExtensionPoint(name, context)` | Executes all handlers registered for a named extension point, sorted by priority, returns results array |
| `getCoreVersion()` | Returns core version for compatibility checking |
| `initialize()` / `shutdown()` | Lifecycle management; shutdown unloads plugins in reverse load order |

**Dependency & Compatibility**:
- Plugins declare `dependencies` (required plugin IDs that must be loaded first)
- `minCoreVersion` / `maxCoreVersion` constrain which core versions a plugin supports
- Loading fails with `PluginError` if dependencies are unmet or version is incompatible

**Events Emitted**:
- `plugin:loaded` — after successful initialization
- `plugin:unloaded` — after shutdown
- `plugin:error` — on any plugin error

### Plugin Interface

```typescript
interface Plugin {
  id: string;              // Unique identifier
  name: string;            // Human-readable name
  version: string;         // Semantic version
  description?: string;
  author?: string;
  homepage?: string;
  priority?: number;       // Execution order (lower = first)
  dependencies?: string[]; // Required plugin IDs
  configSchema?: Record<string, unknown>;
  minCoreVersion?: string;
  maxCoreVersion?: string;

  initialize(config?: Record<string, unknown>): Promise<void>;
  shutdown(): Promise<void>;
  getExtensionPoints(): ExtensionPoint[];
}
```

### ExtensionPoint Interface

```typescript
interface ExtensionPoint {
  name: string;                              // Dot-notation name (e.g., 'workflow.beforeExecute')
  handler: (context: unknown) => Promise<unknown>; // Handler function
  priority?: number;                         // Execution priority
}
```

Extension points are the primary mechanism for plugins to inject behavior. The WorkflowEngine, for example, invokes `workflow.beforeExecute` and `workflow.afterExecute` extension points, allowing plugins to intercept and augment workflow execution.

---

## Plugin SDK (`@claude-flow/plugins`)

The SDK provides builder APIs for creating plugins:

- **PluginBuilder**: Fluent builder for assembling plugins with MCP tools, hooks, workers, providers
- **MCPToolBuilder**: Build MCP tools with typed parameters (string, number, boolean, enum)
- **HookBuilder**: Build hooks with conditions, priorities, and data transformers
- **WorkerPool**: Managed worker pool with auto-scaling (min/max workers, task queuing)
- **ProviderRegistry**: LLM provider management with fallback and cost optimization
- **AgentDBBridge**: Vector storage with HNSW indexing (150x faster search, batch operations)

**Performance Targets**: Plugin load <20ms, hook execution <0.5ms, worker spawn <50ms

---

## Available Plugins (15 Domain-Specific)

### Quality & Testing
| Plugin | Description | MCP Tools |
|--------|-------------|-----------|
| **agentic-qe** | Quality Engineering with 58 AI agents across 13 DDD contexts. London-style TDD, O(log n) coverage gap detection. | 16 tools: `aqe/generate-tests`, `aqe/tdd-cycle`, `aqe/analyze-coverage`, `aqe/security-scan`, `aqe/chaos-inject` |

### AI Interpretability
| Plugin | Description | MCP Tools |
|--------|-------------|-----------|
| **prime-radiant** | Mathematical AI interpretability with 6 engines: sheaf cohomology, spectral analysis, causal inference, quantum topology, category theory, HoTT proofs. | 6 tools: `pr_coherence_check`, `pr_spectral_analyze`, `pr_causal_infer`, `pr_consensus_verify`, `pr_quantum_topology`, `pr_memory_gate` |

### Orchestration
| Plugin | Description | MCP Tools |
|--------|-------------|-----------|
| **gastown-bridge** | Gas Town orchestrator with WASM-accelerated formula parsing (352x faster), Beads sync, convoy management, graph analysis. | 20 tools |
| **teammate-plugin** | Native TeammateTool for Claude Code v2.1.19+. BMSSP WASM, rate limiting, circuit breaker, semantic routing. | 21 tools: `teammate/spawn`, `teammate/coordinate`, `teammate/broadcast`, `teammate/route-task` |

### Domain-Specific (Healthcare, Finance, Legal)
| Plugin | Compliance | Capabilities |
|--------|-----------|--------------|
| **healthcare-clinical** | HIPAA-compliant | FHIR/HL7 integration, symptom analysis, drug interactions, treatment recommendations |
| **financial-risk** | PCI-DSS/SOX | Portfolio optimization, fraud detection, regulatory compliance, market simulation |
| **legal-contracts** | Attorney-client privilege | Contract analysis, risk identification, clause extraction, compliance verification |

### Development Intelligence
| Plugin | Capabilities |
|--------|-------------|
| **code-intelligence** | GNN-based pattern recognition, security vulnerability detection, refactoring suggestions, architecture analysis |
| **test-intelligence** | AI-powered test generation, coverage analysis, mutation testing, test prioritization, flaky test detection |
| **perf-optimizer** | Performance profiling, memory leak detection, CPU bottleneck analysis, I/O optimization, caching strategies |

### Advanced AI / Reasoning
| Plugin | Capabilities |
|--------|-------------|
| **neural-coordination** | Multi-agent neural coordination with SONA learning, agent specialization, knowledge transfer, collective decision making |
| **cognitive-kernel** | Working memory, attention control, meta-cognition, task scaffolding. Miller's Law (7±2) compliance |
| **quantum-optimizer** | Quantum-inspired optimization (QAOA, VQE, quantum annealing), combinatorial optimization, Grover search, tensor networks |
| **hyperbolic-reasoning** | Hyperbolic geometry for hierarchical reasoning, Poincaré embeddings, tree-like structure analysis, taxonomic inference |

---

## Plugin Hook Events (25+)

Plugins can register handlers for any of these lifecycle events:

| Category | Events |
|----------|--------|
| **Session** | `session:start`, `session:end` |
| **Agent** | `agent:pre-spawn`, `agent:post-spawn`, `agent:pre-terminate` |
| **Task** | `task:pre-execute`, `task:post-complete`, `task:error` |
| **Tool** | `tool:pre-call`, `tool:post-call` |
| **Memory** | `memory:pre-store`, `memory:post-store`, `memory:pre-retrieve` |
| **Swarm** | `swarm:initialized`, `swarm:shutdown`, `swarm:consensus-reached` |
| **File** | `file:pre-read`, `file:post-read`, `file:pre-write` |
| **Learning** | `learning:pattern-learned`, `learning:pattern-applied` |

---

## Plugin Marketplace

- **Distribution**: Decentralized via IPFS/Pinata — immutable, content-addressed
- **Registry**: JSON metadata stored on IPFS with CID reference in `discovery.ts`
- **Discovery**: Categories, tags, search, featured/trending lists
- **Security**: SHA-256 checksums, verified authors, trust levels (official/community)
- **CLI Integration**: `npx @claude-flow/cli plugins list`, `install`, `uninstall`, `enable`, `disable`

---

## Design Decisions

| Decision | Rationale |
|----------|-----------|
| Microkernel pattern (ADR-004) | Keeps core minimal; all features are pluggable |
| Version compatibility checks | Prevents breaking plugins on core upgrades |
| Priority-based extension points | Allows ordered execution when multiple plugins extend the same point |
| IPFS marketplace | Decentralized distribution; no single point of failure |
| Domain-specific plugins | Demonstrates enterprise extensibility for healthcare, finance, legal |
| Builder APIs | Fluent builder pattern reduces boilerplate for plugin authors |
| Hot-reload support | Enables development iteration without restarting the system |
