# Agent Definitions

## Overview

Claude Flow ships with **60+ pre-defined specialized agent types** covering the full spectrum of software engineering tasks. Agents are defined as YAML configuration files and are instantiated at runtime by the SwarmCoordinator. A 3-tier model routing system assigns each agent to the optimal LLM tier based on task complexity to minimize cost and latency.

**Key Locations**:

| Path | Description |
|------|-------------|
| `agents/` (root) | Core agent YAML definitions |
| `v3/agents/` | V3 agent YAML definitions |
| `v3/@claude-flow/cli/agents/` | CLI-packaged agent definitions |

---

## YAML Agent Definitions

Core agents are defined as YAML files:

| File | Agent Type | Role |
|------|-----------|------|
| `architect.yaml` | System Architect | System design, architecture decisions, component decomposition |
| `coder.yaml` | Coder | Code implementation, refactoring, debugging |
| `reviewer.yaml` | Reviewer | Code review, quality analysis, security audit |
| `tester.yaml` | Tester | Test writing, validation, coverage analysis |
| `security-architect.yaml` | Security Architect | Security architecture, threat modeling, vulnerability assessment |

Each YAML file defines the agent's capabilities, default role, system prompt, and behavioral constraints.

---

## Full Agent Catalog (60+ Types)

### Core Development Agents

| Agent | Capabilities | Typical Tasks |
|-------|-------------|---------------|
| `coder` | code, refactor, debug | Feature implementation, bug fixes, code changes |
| `reviewer` | review, analyze, security-audit | Code review, quality analysis, best practices |
| `tester` | test, validate, e2e | Unit tests, integration tests, E2E tests |
| `planner` | plan, decompose, estimate | Task breakdown, sprint planning, estimation |
| `researcher` | research, analyze, document | Requirements analysis, technology research |

### V3 Specialized Agents

| Agent | Specialization |
|-------|---------------|
| `security-architect` | Security architecture, threat modeling |
| `security-auditor` | Security scanning, vulnerability detection |
| `memory-specialist` | Memory optimization, caching strategies |
| `performance-engineer` | Performance profiling, bottleneck analysis |

### Swarm Coordination Agents

| Agent | Topology | Role |
|-------|----------|------|
| `hierarchical-coordinator` | Hierarchical | Queen/leader that delegates to workers |
| `mesh-coordinator` | Mesh | Peer coordinator in fully connected mesh |
| `adaptive-coordinator` | Adaptive | Dynamically switches coordination strategy |
| `collective-intelligence-coordinator` | Any | Aggregates collective agent intelligence |
| `swarm-memory-manager` | Any | Manages shared swarm memory |

### Consensus & Distributed Agents

| Agent | Algorithm | Purpose |
|-------|-----------|---------|
| `byzantine-coordinator` | Byzantine BFT | Fault-tolerant consensus (f < n/3) |
| `raft-manager` | Raft | Leader-based consensus (f < n/2) |
| `gossip-coordinator` | Gossip | Epidemic protocol for eventual consistency |
| `consensus-builder` | Any | Generic consensus facilitation |
| `crdt-synchronizer` | CRDT | Conflict-free replicated data synchronization |
| `quorum-manager` | Quorum | Configurable quorum-based decisions |
| `security-manager` | — | Security policy enforcement in distributed systems |

### Performance & Optimization Agents

| Agent | Focus |
|-------|-------|
| `perf-analyzer` | Performance analysis and profiling |
| `performance-benchmarker` | Benchmark creation and execution |
| `task-orchestrator` | Task scheduling and optimization |
| `memory-coordinator` | Memory allocation and GC optimization |
| `smart-agent` | Self-optimizing agent with learning |

### GitHub & Repository Agents

| Agent | Focus |
|-------|-------|
| `github-modes` | GitHub API integration modes |
| `pr-manager` | Pull request creation, review, merge |
| `code-review-swarm` | Multi-agent code review |
| `issue-tracker` | Issue triage and management |
| `release-manager` | Release planning and execution |
| `workflow-automation` | CI/CD workflow management |
| `project-board-sync` | Project board synchronization |
| `repo-architect` | Repository structure and architecture |
| `multi-repo-swarm` | Cross-repository coordination |

### SPARC Methodology Agents

| Agent | SPARC Phase |
|-------|------------|
| `sparc-coord` | Overall SPARC coordination |
| `sparc-coder` | SPARC-guided implementation |
| `specification` | **S**pecification phase |
| `pseudocode` | **P**seudocode phase |
| `architecture` | **A**rchitecture phase |
| `refinement` | **R**efinement phase |

### Specialized Development Agents

| Agent | Domain |
|-------|--------|
| `backend-dev` | Backend/API development |
| `mobile-dev` | Mobile application development |
| `ml-developer` | Machine learning development |
| `cicd-engineer` | CI/CD pipeline engineering |
| `api-docs` | API documentation generation |
| `system-architect` | Full system architecture |
| `code-analyzer` | Static code analysis |
| `base-template-generator` | Project template generation |

### Testing & Validation Agents

| Agent | Approach |
|-------|----------|
| `tdd-london-swarm` | London-style TDD with multi-agent swarm |
| `production-validator` | Production readiness validation |

---

## 3-Tier Model Routing (ADR-026)

Agents are routed to the optimal LLM tier based on task complexity analysis:

| Tier | Handler | Latency | Cost | Use Cases |
|------|---------|---------|------|-----------|
| **1** | Agent Booster (WASM) | <1ms | $0 | Simple transforms: var→const, add types, remove console.log, add error handling, async→await |
| **2** | Haiku | ~500ms | $0.0002 | Simple tasks, bug fixes, low complexity (<30% score) |
| **3** | Sonnet / Opus | 2-5s | $0.003-$0.015 | Architecture, security, complex reasoning (>30% score) |

**Cost Impact**: 75% reduction through intelligent routing. Tier 1 tasks (which are common in large codebases) skip LLM calls entirely.

### Routing Detection

```bash
# Get routing recommendation before spawning
npx @claude-flow/cli hooks pre-task --description "[task description]"
# Returns: [AGENT_BOOSTER_AVAILABLE] or [TASK_MODEL_RECOMMENDATION] Use model="haiku|sonnet|opus"
```

---

## Agent Routing by Task Type

| Task Type | Required Agents | Topology | Model Tier |
|-----------|----------------|----------|------------|
| Bug Fix | researcher, coder, tester | mesh | Tier 2-3 |
| New Feature | coordinator, architect, coder, tester, reviewer | hierarchical | Tier 3 |
| Refactoring | architect, coder, reviewer | mesh | Tier 2-3 |
| Performance | researcher, perf-engineer, coder | hierarchical | Tier 3 |
| Security Audit | security-architect, auditor | hierarchical | Tier 3 |
| Memory Optimization | memory-specialist, perf-engineer | mesh | Tier 2-3 |
| Documentation | researcher, api-docs | mesh | Tier 2 |

---

## Agent Capabilities Model

Each agent defines a set of **capabilities** — strings that describe what it can do. The SwarmCoordinator uses capabilities to match tasks to suitable agents:

```typescript
// Capability → Task Type mapping
const typeToCapability = {
  code: 'code',
  test: 'test',
  review: 'review',
  design: 'design',
  deploy: 'deploy',
  refactor: 'refactor',
  debug: 'debug',
};

// Default capabilities per agent type
const defaults = {
  coder: ['code', 'refactor', 'debug'],
  tester: ['test', 'validate', 'e2e'],
  reviewer: ['review', 'analyze', 'security-audit'],
  coordinator: ['coordinate', 'manage', 'orchestrate'],
  designer: ['design', 'prototype'],
  deployer: ['deploy', 'release'],
};
```

---

## Design Decisions

| Decision | Rationale |
|----------|-----------|
| YAML agent definitions | Human-readable, easy to customize, version-controllable |
| 60+ pre-built agents | Comprehensive coverage eliminates manual agent design |
| 3-tier model routing | Optimizes cost by routing simple tasks to cheaper/faster tiers |
| Capability-based matching | Flexible assignment without hardcoded task→agent maps |
| Task-type routing tables | Deterministic agent selection for known task patterns |
| Extensible type unions (`| string`) | Custom agent types without modifying core code |
| SPARC methodology agents | Structured development methodology support |
| Consensus-specific agents | Distributed systems patterns built into the agent catalog |
