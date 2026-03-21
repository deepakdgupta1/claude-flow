# Swarm Coordination Module

## Overview

The Swarm Coordination module manages multi-agent orchestration — spawning agents, distributing tasks, managing topologies, inter-agent messaging, and consensus. It is the central nervous system of Claude Flow's multi-agent architecture.

**Locations**:
- `v3/src/coordination/application/SwarmCoordinator.ts` — Core coordinator
- `v3/@claude-flow/swarm/` — Full swarm package with advanced coordination

---

## Core SwarmCoordinator

**Location**: `v3/src/coordination/application/SwarmCoordinator.ts`

The `SwarmCoordinator` manages a pool of agents, distributes work across them, and maintains topology-aware connections.

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `topology` | `SwarmTopology` | One of `'hierarchical'`, `'mesh'`, `'simple'`, `'adaptive'` |
| `agents` | `Map<string, Agent>` | Active agent pool keyed by agent ID |
| `memoryBackend` | `IMemoryBackend` | Persistence layer for events and metrics |
| `eventBus` | `EventBus` | Event bus for lifecycle and messaging events |
| `pluginManager` | `PluginManager` | Plugin system for extension points |
| `agentMetrics` | `Map<string, AgentMetrics>` | Per-agent performance metrics |
| `connections` | `MeshConnection[]` | Topology connections between agents |

### Agent Lifecycle

#### `spawnAgent(config: AgentConfig): Agent`

Creates a new agent and integrates it into the swarm.

**Flow**:
1. Creates a new `Agent` instance from the provided config.
2. Initializes agent metrics:
   - `tasksCompleted`: 0
   - `successRate`: 1.0
   - `health`: `'healthy'`
3. Creates topology-aware connections to existing agents (see [Topology Connections](#topology-connections)).
4. Emits an `'agent:spawned'` event via the event bus.
5. Stores the spawn event in the memory backend.
6. Returns the new `Agent` instance.

#### `terminateAgent(agentId: string): void`

Removes an agent from the swarm.

**Flow**:
1. Calls `agent.terminate()` to set the agent's status.
2. Removes all connections involving this agent.
3. Emits an `'agent:terminated'` event.

#### `scaleAgents(options: { type: AgentType, count: number }): void`

Scales the number of agents of a given type up or down. When scaling up, new agents are spawned with the specified type. When scaling down, excess agents of that type are terminated (least recently active first).

### Task Distribution

#### `distributeTasks(tasks: Task[]): Map<string, Task[]>`

Load-balanced task distribution across available agents.

**Algorithm**:
1. Sorts tasks by priority (high → low).
2. For each task, finds suitable agents by matching the task type to agent capabilities (via `agent.canExecute()`).
3. Among suitable agents, selects the one with the **lowest current load** (fewest in-progress tasks).
4. Assigns the task to the selected agent.
5. Returns a map of agent IDs to their assigned task arrays.

#### `executeTask(agentId: string, task: Task): Promise<TaskResult>`

Executes a specific task on a specific agent.

**Flow**:
1. Retrieves the agent from the pool.
2. Calls `agent.executeTask(task)`.
3. Updates agent metrics:
   - Increments `tasksCompleted`
   - Recalculates `successRate`
   - Updates `averageExecutionTime`
4. Stores the execution result in the memory backend.
5. Returns the `TaskResult`.

#### `executeTasksConcurrently(tasks: Task[]): Promise<TaskResult[]>`

Distributes tasks using `distributeTasks()`, then executes all assignments in parallel using `Promise.all`. Returns an array of all task results.

### Communication

#### `sendMessage(message: AgentMessage): void`

Sends a message between agents via the event bus. Messages are typed and can carry arbitrary payloads. The event bus routes messages to the target agent(s) based on the message's recipient field.

### Swarm State

#### `getSwarmState(): SwarmState`

Returns a snapshot of the current swarm:
- `agents`: List of all agents with their current status
- `topology`: Current topology type
- `leader`: ID of the leader agent (if hierarchical)
- `connectionCount`: Number of active connections

### Consensus

#### `reachConsensus(decision: string, agentIds: string[]): Promise<ConsensusResult>`

Simple majority voting across the specified agents. Each agent casts a vote, and the decision passes if more than half agree. Returns the result with vote counts.

### Dependency Resolution

#### `resolveTaskDependencies(tasks: Task[]): Task[]`

Delegates to `Task.resolveExecutionOrder()` (see [02-task-execution.md](./02-task-execution.md)) for topological sorting and circular dependency detection.

### Topology Connections

Connection patterns are determined by the `topology` setting:

| Topology | Connection Pattern |
|----------|-------------------|
| `mesh` | All-to-all peer connections. Every agent is connected to every other agent. |
| `hierarchical` | Workers connect to the leader agent. Leader coordinates all workers. |
| `simple` | No explicit connections. Tasks are distributed centrally. |
| `adaptive` | Starts as simple, dynamically adds connections based on communication patterns. |

---

## @claude-flow/swarm Package

**Location**: `v3/@claude-flow/swarm/`

The full swarm package provides advanced coordination capabilities beyond the core `SwarmCoordinator`.

### UnifiedSwarmCoordinator

The canonical coordination engine that consolidates four legacy coordination systems into a single, optimized implementation.

**Key characteristics**:
- Supports up to 15-agent hierarchical mesh topologies
- Coordination latency: <100ms
- Consensus latency: <100ms
- Message throughput: 1000+ messages/second

### QueenCoordinator

Central hive-mind orchestrator that provides intelligent, strategic coordination.

**Capabilities**:

| Capability | Description | Performance |
|------------|-------------|-------------|
| Strategic task analysis | Uses ReasoningBank pattern matching to analyze incoming tasks and determine optimal execution strategies | <50ms |
| Agent delegation | Scores agents across five dimensions for optimal assignment | <20ms |
| Health monitoring | Continuous monitoring with bottleneck detection and auto-recovery | <30ms |
| Consensus coordination | Supports multiple consensus modes | <100ms |
| Learning from outcomes | Records neural trajectories to improve future decisions | — |

**Agent Scoring Dimensions**:
1. **Capability score**: Does the agent have the required capabilities?
2. **Load score**: How busy is the agent currently?
3. **Performance score**: What is the agent's historical success rate?
4. **Health score**: Is the agent healthy, degraded, or unhealthy?
5. **Availability score**: Is the agent available to accept new work?

**Consensus Modes**:
- `majority`: >50% agreement
- `supermajority`: >66% agreement
- `unanimous`: 100% agreement
- `weighted`: Votes weighted by agent role and performance

### TopologyManager

Manages the structural relationships between agents in the swarm.

**Supported Topologies**:
- `mesh`: Fully connected peer network
- `hierarchical`: Tree structure with leader at root
- `centralized`: Single coordinator, all workers connect to center
- `hybrid`: Combination of hierarchical and mesh patterns

**Features**:
- **O(1) role-based indexes**: Instant lookup of queen, coordinator, and other role-specific agents.
- **Auto-rebalance**: Automatically redistributes connections when agents join or leave, with flood prevention to avoid cascading rebalances.
- **Partition management**: Supports `hash` and `range` partitioning strategies for distributing workload segments.
- **Leader election**: Based on role priority — higher-priority roles (queen > coordinator > worker) take precedence in elections.

### MessageBus

Inter-agent communication layer. Handles message routing, delivery guarantees, and pub/sub patterns. Agents can send point-to-point messages or broadcast to groups.

### AgentPool

Managed pool of agents with workload balancing. Tracks agent availability, automatically assigns idle agents to pending tasks, and handles pool scaling.

### ConsensusEngine

Pluggable consensus algorithms for distributed decision-making.

| Algorithm | Fault Tolerance | Description |
|-----------|----------------|-------------|
| **Raft** | f < n/2 | Leader-based consensus. The leader proposes decisions, followers acknowledge. Tolerates up to half the nodes failing. |
| **Byzantine** (BFT) | f < n/3 | Byzantine fault tolerant. Handles malicious or arbitrary failures. Tolerates up to one-third of nodes being faulty. |
| **Gossip** | Eventual | Epidemic protocol for eventual consistency. Each node periodically shares state with random peers. Best for large swarms where strong consistency is not required. |

### FederationHub

Enables ephemeral agent coordination across swarm boundaries. Agents from different swarms can temporarily collaborate on shared tasks without permanent topology changes.

### AttentionCoordinator

Applies attention mechanisms from neural network architectures to agent coordination:
- **Flash Attention**: Optimized attention computation for fast coordination decisions
- **Mixture of Experts (MoE)**: Routes coordination decisions to specialized sub-coordinators
- **GraphRoPE**: Graph-aware rotary position embeddings for topology-aware attention

### WorkerDispatchService

Background worker dispatch service compatible with the agentic-flow runtime. Manages long-running background workers that perform continuous tasks (monitoring, optimization, analysis) alongside the main swarm operations.

---

## Default Configuration

| Setting | Default Value |
|---------|---------------|
| Topology | `hierarchical` |
| Max agents | 15 |
| Consensus algorithm | `raft` |
| Consensus threshold | 0.66 (supermajority) |
| Consensus timeout | 5 seconds |

---

## Design Decisions

### Hierarchical as Default Topology

The hierarchical topology is the default because it provides clear coordination authority and prevents drift. A single leader validates each output against the goal, catching divergence early. For scenarios requiring peer-to-peer communication, `mesh` or `hybrid` topologies are available.

### Load-Based Task Distribution

Task distribution uses a load-balancing strategy rather than round-robin or random assignment. This ensures that busy agents are not overloaded while idle agents sit unused. The algorithm considers both capability matching and current load.

### Separation of Core and Package

The core `SwarmCoordinator` in `v3/src/coordination/` provides the essential coordination primitives. The `@claude-flow/swarm` package extends these with advanced features (QueenCoordinator, ConsensusEngine, FederationHub). This separation allows lightweight deployments that only need basic coordination without pulling in the full swarm package.

### Metrics-Driven Agent Management

Every task execution updates agent metrics (success rate, execution time, health). These metrics feed back into task distribution decisions, consensus weighting, and health monitoring. Poor-performing agents receive fewer tasks, and unhealthy agents trigger alerts.

---

## Usage Example

```typescript
import { SwarmCoordinator } from './application/SwarmCoordinator';

// Initialize coordinator
const coordinator = new SwarmCoordinator({
  topology: 'hierarchical',
  memoryBackend,
  eventBus,
});

// Spawn agents
const leader = coordinator.spawnAgent({
  id: 'leader-01',
  type: 'coordinator',
  capabilities: ['code', 'review'],
  role: 'leader',
});

const coder = coordinator.spawnAgent({
  id: 'coder-01',
  type: 'coder',
  capabilities: ['code', 'refactor', 'debug'],
  role: 'worker',
  parent: 'leader-01',
});

const tester = coordinator.spawnAgent({
  id: 'tester-01',
  type: 'tester',
  capabilities: ['test'],
  role: 'worker',
  parent: 'leader-01',
});

// Distribute and execute tasks
const tasks = [codeTask, testTask, reviewTask];
const assignments = coordinator.distributeTasks(tasks);

// Execute concurrently
const results = await coordinator.executeTasksConcurrently(tasks);

// Check swarm state
const state = coordinator.getSwarmState();
// { agents: [...], topology: 'hierarchical', leader: 'leader-01', connectionCount: 2 }

// Scale up
coordinator.scaleAgents({ type: 'coder', count: 3 });

// Consensus
const decision = await coordinator.reachConsensus(
  'Should we deploy to production?',
  ['leader-01', 'coder-01', 'tester-01']
);
```
