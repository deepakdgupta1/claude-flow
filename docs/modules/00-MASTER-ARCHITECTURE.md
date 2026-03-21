# Claude Flow V3 — Master Architecture Document

## 1. Executive Summary

**Claude Flow** is an enterprise-grade, open-source AI agent orchestration platform that transforms Claude Code into a multi-agent development environment. It enables teams to deploy, coordinate, and optimize **60+ specialized AI agents** working together on complex software engineering tasks through coordinated swarms with self-learning capabilities, fault-tolerant consensus, and enterprise-grade security.

| Attribute | Value |
|-----------|-------|
| **Version** | 3.0.0-alpha (V3) |
| **License** | MIT |
| **Runtime** | Node.js ≥20.0.0 |
| **Language** | TypeScript (ES2022, ESM) |
| **Primary Interface** | CLI (26 commands, 140+ subcommands) + MCP Server |
| **Integration** | Claude Code via Model Context Protocol (MCP) |
| **npm Package** | `claude-flow` / `@claude-flow/cli` |
| **Repository** | https://github.com/ruvnet/claude-flow |

---

## 2. Problem Statement & Targeted Use Cases

### Problem
Modern software engineering tasks — feature implementation, refactoring, security audits, performance optimization — are complex enough to benefit from multiple specialized AI perspectives working simultaneously. A single AI assistant processes tasks sequentially, lacks persistent memory across sessions, and cannot coordinate subtasks in parallel.

### Targeted Use Cases

| Use Case | How Claude Flow Addresses It |
|----------|------------------------------|
| **Multi-file feature implementation** | Spawns architect + coder + tester + reviewer agents in parallel with a hierarchical coordinator |
| **Codebase-wide refactoring** | Mesh topology agents analyze and transform code concurrently, coordinator prevents drift |
| **Security audits** | Security-architect + auditor agents scan for vulnerabilities with OWASP/SANS compliance |
| **Performance optimization** | Performance-engineer agents profile, identify bottlenecks, implement fixes with benchmarking |
| **Cross-session learning** | ReasoningBank stores successful patterns; SONA adapts routing based on what worked |
| **Enterprise compliance** | Domain-specific plugins for healthcare (HIPAA), finance (PCI-DSS), legal (attorney-client privilege) |
| **Cost optimization** | 3-tier model routing sends simple tasks to cheap/free tiers; 75% cost reduction |
| **Air-gapped deployment** | Ollama provider support enables fully local AI model usage |

---

## 3. High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        USER LAYER                               │
│   Claude Code ←──MCP──→ CLI (auto-detects MCP mode via stdin)   │
│                         Terminal (interactive CLI)               │
└──────────────────────────────┬──────────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────────┐
│                     ROUTING LAYER                                │
│   Q-Learning Router │ MoE (8 Experts) │ 3-Tier Model Routing    │
│   Skills (42+)      │ Hooks (27)      │ Agent Booster (WASM)    │
│   AIDefence (prompt injection prevention)                       │
└──────────────────────────────┬──────────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────────┐
│                   COORDINATION LAYER                             │
│   SwarmCoordinator │ UnifiedSwarmCoordinator │ QueenCoordinator  │
│   TopologyManager (mesh/hierarchical/centralized/hybrid)         │
│   ConsensusEngine (Raft/Byzantine/Gossip)                        │
│   MessageBus │ AgentPool │ FederationHub │ AttentionCoordinator  │
└──────────────────────────────┬──────────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────────┐
│                      AGENT LAYER (60+ types)                     │
│   coder │ tester │ reviewer │ architect │ security-architect     │
│   hierarchical-coordinator │ byzantine-coordinator │ ...         │
│   Capabilities: code, test, review, design, deploy, refactor    │
└──────────────────────────────┬──────────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────────┐
│                    RESOURCE LAYER                                │
│   Memory: SQLite + AgentDB (HNSW) = HybridBackend              │
│   Providers: Anthropic │ OpenAI │ Google │ Cohere │ Ollama      │
│   Workers: 12 background (ultralearn, audit, optimize, ...)     │
│   Plugins: 15 domain-specific + IPFS marketplace                │
└──────────────────────────────┬──────────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────────┐
│                  INTELLIGENCE LAYER (RuVector)                   │
│   SONA (<0.05ms adaptation) │ EWC++ (no forgetting)             │
│   HNSW (150x-12,500x faster) │ ReasoningBank (pattern store)   │
│   Flash Attention (2.49-7.47x) │ LoRA/MicroLoRA (128x compress)│
│   9 RL Algorithms │ Hyperbolic Embeddings (Poincaré)            │
│                                                                  │
│   Pipeline: RETRIEVE → JUDGE → DISTILL → CONSOLIDATE            │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. Module Map

The codebase is organized into distinct modules, each documented in a separate file:

| # | Module | Location | Doc File |
|---|--------|----------|----------|
| 01 | **Agent Lifecycle** | `v3/src/agent-lifecycle/` | [01-agent-lifecycle.md](01-agent-lifecycle.md) |
| 02 | **Task Execution & Workflows** | `v3/src/task-execution/` | [02-task-execution.md](02-task-execution.md) |
| 03 | **Swarm Coordination** | `v3/src/coordination/` + `v3/@claude-flow/swarm/` | [03-swarm-coordination.md](03-swarm-coordination.md) |
| 04 | **Memory System** | `v3/src/memory/` + `v3/@claude-flow/memory/` + `v3/@claude-flow/embeddings/` | [04-memory-system.md](04-memory-system.md) |
| 05 | **Neural Learning** | `v3/@claude-flow/neural/` | [05-neural-learning.md](05-neural-learning.md) |
| 06 | **Security** | `v3/@claude-flow/security/` + `v3/@claude-flow/aidefence/` | [06-security.md](06-security.md) |
| 07 | **Hooks System** | `v3/@claude-flow/hooks/` | [07-hooks-system.md](07-hooks-system.md) |
| 08 | **Providers** | `v3/@claude-flow/providers/` | [08-providers.md](08-providers.md) |
| 09 | **Plugin System** | `v3/src/infrastructure/plugins/` + `v3/@claude-flow/plugins/` + `v3/plugins/` | [09-plugin-system.md](09-plugin-system.md) |
| 10 | **MCP Server** | `v3/mcp/` + `v3/src/infrastructure/mcp/` | [10-mcp-server.md](10-mcp-server.md) |
| 11 | **CLI** | `v3/@claude-flow/cli/` | [11-cli.md](11-cli.md) |
| 12 | **Shared Types** | `v3/src/shared/types/` | [12-shared-types.md](12-shared-types.md) |
| 13 | **Claims Authorization** | `v3/@claude-flow/claims/` | [13-claims-authorization.md](13-claims-authorization.md) |
| 14 | **V2 Legacy** | `v2/` | [14-v2-legacy.md](14-v2-legacy.md) |
| 15 | **Agent Definitions** | `agents/` + `v3/agents/` | [15-agent-definitions.md](15-agent-definitions.md) |

---

## 5. Package Structure

### Root Level

```
claude-flow/
├── agents/              # Core agent YAML definitions
├── docs/                # Documentation (this file)
├── plugin/              # Claude Code plugin integration
├── scripts/             # Install scripts (install.sh, cleanup)
├── tests/               # Docker regression tests
├── v2/                  # V2 legacy codebase
├── v3/                  # V3 main codebase (active)
├── package.json         # Root package (claude-flow npm package)
├── tsconfig.json        # Root TypeScript config
└── CLAUDE.md            # Claude Code agent instructions
```

### V3 Codebase (`v3/`)

```
v3/
├── @claude-flow/        # Scoped packages (20 modules)
│   ├── cli/             # CLI + MCP server entry point
│   ├── swarm/           # Swarm coordination
│   ├── neural/          # Neural learning (SONA, RL, ReasoningBank)
│   ├── memory/          # Memory backends (SQLite, AgentDB, Hybrid)
│   ├── embeddings/      # Vector embeddings (chunking, normalization, hyperbolic)
│   ├── hooks/           # Lifecycle hooks + background workers
│   ├── security/        # Security module (CVE fixes, validation, crypto)
│   ├── providers/       # LLM providers (Anthropic, OpenAI, Google, Cohere, Ollama)
│   ├── plugins/         # Plugin SDK + marketplace
│   ├── claims/          # Claims-based authorization
│   ├── aidefence/       # AI-specific security (prompt injection detection)
│   ├── agents/          # Agent package
│   ├── browser/         # Browser automation integration
│   ├── deployment/      # Deployment management
│   ├── integration/     # External integrations
│   ├── mcp/             # MCP package
│   ├── performance/     # Performance profiling
│   ├── shared/          # Shared utilities
│   └── testing/         # Testing utilities
├── src/                 # Core domain model (DDD structure)
│   ├── agent-lifecycle/ # Agent domain entity
│   ├── coordination/    # SwarmCoordinator application service
│   ├── memory/          # Memory domain + infrastructure
│   ├── task-execution/  # Task domain + WorkflowEngine
│   ├── infrastructure/  # MCP + Plugin infrastructure
│   ├── shared/          # Shared types (single source of truth)
│   └── index.ts         # Public API exports
├── mcp/                 # MCP server implementation
├── plugins/             # 15 domain-specific plugins
├── agents/              # Agent YAML definitions
└── __tests__/           # Test suites
```

---

## 6. Data Flow

### Request Lifecycle (CLI → Agent Execution → Memory)

```
1. User enters command / Claude Code calls MCP tool
                    ↓
2. CLI parses command / MCP Server routes request
                    ↓
3. Hooks: pre-task fires → ReasoningBank retrieves patterns
                    ↓
4. Router determines optimal agents + topology + model tier
                    ↓
5. SwarmCoordinator spawns agents, creates topology connections
                    ↓
6. Tasks distributed to agents (load-balanced, priority-sorted)
                    ↓
7. Agents execute tasks (via onExecute callbacks → LLM providers)
                    ↓
8. Results stored in HybridBackend (SQLite + AgentDB)
                    ↓
9. Hooks: post-task fires → patterns stored in ReasoningBank
                    ↓
10. Neural system: RETRIEVE → JUDGE → DISTILL → CONSOLIDATE
                    ↓
11. Response returned to user / Claude Code
```

### Learning Loop

```
Successful Task Completion
        ↓
    Store pattern in ReasoningBank
        ↓
    SONA records trajectory (context → action → reward)
        ↓
    ReasoningBank judges quality
        ↓
    LoRA distills into reusable memory
        ↓
    EWC++ consolidates without forgetting
        ↓
    Next similar task → pattern retrieved → better routing
```

---

## 7. Key Design Decisions & ADRs

| ADR | Decision | Rationale |
|-----|----------|-----------|
| ADR-003 | Unified SwarmCoordinator | Single canonical coordination engine replacing 4 legacy systems |
| ADR-004 | Microkernel plugin architecture | Core stays minimal; all features are pluggable |
| ADR-009 | Hybrid memory backend as default | SQLite for structured queries + AgentDB for vector search |
| ADR-026 | 3-tier model routing | 75% cost reduction by routing simple tasks to cheap/free tiers |
| — | DDD structure in `v3/src/` | Domain/Application/Infrastructure separation |
| — | Callback-based task execution | Agents are lightweight wrappers; business logic lives in task callbacks |
| — | stdio as default MCP transport | Native to Claude Code's MCP integration |
| — | RETRIEVE→JUDGE→DISTILL→CONSOLIDATE pipeline | Mimics human learning from experience |
| — | Claims-based authorization | Flexible enough for RBAC, ABAC, and human-agent coordination |
| — | YAML agent definitions | Human-readable, version-controllable, easily customizable |

---

## 8. Technology Stack

| Layer | Technology |
|-------|-----------|
| **Language** | TypeScript 5.x (ES2022, ESM modules) |
| **Runtime** | Node.js ≥20.0.0 |
| **Build** | tsc (TypeScript compiler) |
| **Test Framework** | Vitest (V3), Jest (V2) |
| **Validation** | Zod |
| **Database** | SQLite (via sql.js WASM — cross-platform, no native deps) |
| **Vector DB** | AgentDB with HNSW indexing |
| **Neural** | RuVector (SONA, EWC++, Flash Attention — Rust/WASM) |
| **Embeddings** | ONNX Runtime (via agentic-flow), MiniLM |
| **Security** | bcrypt, crypto (Node.js built-in), Zod schemas |
| **Transport** | stdio (MCP), HTTP, WebSocket |
| **Plugin Distribution** | IPFS via Pinata |
| **Package Manager** | npm (also supports pnpm) |

---

## 9. Performance Targets

| Metric | Target | Module |
|--------|--------|--------|
| CLI startup | <500ms | CLI |
| MCP server startup | <400ms | MCP Server |
| MCP tool registration | <10ms | MCP Server |
| MCP tool execution overhead | <50ms | MCP Server |
| Agent coordination (15 agents) | <100ms | Swarm |
| Consensus operations | <100ms | Swarm |
| Message throughput | 1000+ msg/sec | Swarm |
| SONA adaptation | <0.05ms | Neural |
| Pattern matching | <1ms | Neural |
| Learning step | <10ms | Neural |
| HNSW vector search | 150x-12,500x faster than brute force | Memory |
| Memory reduction (Int8 quantization) | 50-75% | Neural |
| Flash Attention speedup | 2.49x-7.47x | Neural |
| Plugin load | <20ms | Plugins |
| Hook execution | <0.5ms | Hooks |
| Worker spawn | <50ms | Hooks |
| Tier 1 task execution (Agent Booster) | <1ms | Agent Definitions |

---

## 10. Security Model

### Defense-in-Depth Layers

1. **Input Validation** (Zod schemas at all system boundaries)
2. **AIDefence** (prompt injection detection and prevention)
3. **Path Validation** (traversal prevention, null byte blocking)
4. **Command Injection Prevention** (whitelist-based safe executor)
5. **Credential Security** (bcrypt hashing, secure token generation, no hardcoded defaults)
6. **Claims-Based Authorization** (fine-grained agent permissions)
7. **Plugin Sandboxing** (version compatibility, dependency validation)

### CVE Remediations Implemented
- CVE-2: Weak password hashing → bcrypt with ≥12 rounds
- CVE-3: Hardcoded credentials → cryptographic credential generation
- HIGH-1: Command injection → whitelist-based SafeExecutor
- HIGH-2: Path traversal → PathValidator with allowed prefix enforcement

---

## 11. Extensibility Model

```
Core (minimal) → Plugins (domain features) → Marketplace (community)
     ↓                    ↓                         ↓
  PluginManager     15 built-in plugins      IPFS distribution
  ExtensionPoints   50+ MCP tools            SHA-256 verified
  Hook Events       Domain-specific          Trust levels
```

### Extension Mechanisms
1. **Plugins**: Full lifecycle with SDK, builder API, MCP tools, hooks, workers
2. **Hook Events**: 25+ lifecycle events (session, agent, task, memory, swarm, file, learning)
3. **Extension Points**: Named injection points in workflow execution
4. **Custom Agents**: YAML definitions with custom capabilities
5. **Custom Providers**: New LLM providers via BaseProvider abstraction
6. **Background Workers**: 12 built-in + custom worker registration

---

## 12. Versioning & Migration

### Current State
- **V3**: Active development (alpha), all new features
- **V2**: Legacy, maintained for backward compatibility
- **Migration**: `npx @claude-flow/cli migrate` (status → run → validate → rollback)

### V2 Compatibility
- `pre-bash`/`post-bash` → `pre-command`/`post-command` (hook aliases)
- `route-task` → `route` (hook alias)
- `SwarmHub` → `UnifiedSwarmCoordinator` (deprecated compatibility layer)
- Session format bridging in `session-start` hook

---

## 13. How to Navigate This Documentation

1. **Start here** (this document) for the overall picture
2. Read [12-shared-types.md](12-shared-types.md) to understand the type system
3. Read [01-agent-lifecycle.md](01-agent-lifecycle.md) → [02-task-execution.md](02-task-execution.md) → [03-swarm-coordination.md](03-swarm-coordination.md) for the core execution flow
4. Read [04-memory-system.md](04-memory-system.md) → [05-neural-learning.md](05-neural-learning.md) for the learning/memory subsystem
5. Read [06-security.md](06-security.md) for security architecture
6. Read [07-hooks-system.md](07-hooks-system.md) for the event/hook system
7. Read [08-providers.md](08-providers.md) for LLM provider integration
8. Read [09-plugin-system.md](09-plugin-system.md) for extensibility
9. Read [10-mcp-server.md](10-mcp-server.md) → [11-cli.md](11-cli.md) for the user-facing interfaces
10. Read [15-agent-definitions.md](15-agent-definitions.md) for the agent catalog and routing
11. Read [13-claims-authorization.md](13-claims-authorization.md) and [14-v2-legacy.md](14-v2-legacy.md) for supplementary topics
