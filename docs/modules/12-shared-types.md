# Module 12: Shared Types

**Location**: `v3/src/shared/types/index.ts`

**Purpose**: Central type definitions used across all V3 modules. Provides the type contracts that bind all components together — agents, tasks, workflows, memory, coordination, plugins, MCP, and error handling all reference types defined here.

---

## Agent Types

### AgentStatus

```typescript
type AgentStatus = 'active' | 'idle' | 'busy' | 'terminated' | 'error';
```

Represents the lifecycle state of an agent. Transitions follow a defined state machine:
- `idle` → `active` → `busy` → `idle` (normal work cycle)
- Any state → `error` (on failure)
- Any state → `terminated` (on shutdown)

### AgentRole

```typescript
type AgentRole = 'leader' | 'worker' | 'peer';
```

Determines an agent's position in the coordination hierarchy:
- **leader**: Coordinates other agents, makes delegation decisions
- **worker**: Executes tasks assigned by a leader
- **peer**: Equal participant in mesh topologies

### AgentType

```typescript
type AgentType = 'coder' | 'tester' | 'reviewer' | 'coordinator' | 'designer' | 'deployer' | string;
```

Identifies the agent's specialization. The `| string` extension allows custom agent types without modifying core type definitions. Built-in types cover the most common software engineering roles.

### AgentCapability

```typescript
interface AgentCapability {
  name: string;
  level?: 'basic' | 'intermediate' | 'advanced';
}
```

Describes a specific skill or capability an agent possesses. The optional `level` field enables capability-based routing — a task requiring `advanced` TypeScript skills will only be assigned to agents with that capability at the appropriate level.

### AgentConfig

```typescript
interface AgentConfig {
  id: string;
  type: AgentType;
  capabilities?: AgentCapability[];
  role?: AgentRole;
  parent?: string;
  metadata?: Record<string, unknown>;
}
```

Configuration used to spawn a new agent. Only `id` and `type` are required — all other fields have sensible defaults. The `parent` field establishes hierarchy relationships for `hierarchical` topologies.

### Agent Interface

```typescript
interface Agent {
  id: string;
  type: AgentType;
  status: AgentStatus;
  capabilities: AgentCapability[];
  role: AgentRole;
  parent?: string;
  metadata?: Record<string, unknown>;
  executeTask?: (task: Task) => Promise<TaskResult>;
}
```

Full runtime state of an agent. The optional `executeTask` method allows agents to have embedded execution logic, though tasks are more commonly dispatched through the coordination layer.

---

## Task Types

### TaskPriority

```typescript
type TaskPriority = 'high' | 'medium' | 'low';
```

Controls scheduling order. High-priority tasks are dequeued first. Within the same priority level, tasks follow FIFO ordering.

### TaskStatus

```typescript
type TaskStatus = 'pending' | 'in-progress' | 'completed' | 'failed' | 'cancelled';
```

Tracks task lifecycle:
- `pending`: Created but not yet assigned
- `in-progress`: Currently being executed by an agent
- `completed`: Successfully finished
- `failed`: Execution resulted in an error
- `cancelled`: Manually or programmatically aborted

### TaskType

```typescript
type TaskType = 'code' | 'test' | 'review' | 'design' | 'deploy' | 'workflow' | string;
```

Categorizes the nature of the work. Used by the routing system to match tasks to appropriate agents. The `workflow` type indicates a composite task containing sub-tasks. Extensible via `| string`.

### Task Interface

```typescript
interface Task {
  id: string;
  type: TaskType;
  name: string;
  description?: string;
  status: TaskStatus;
  priority: TaskPriority;
  assignee?: string;
  dependencies?: string[];
  metadata?: Record<string, unknown>;
  workflow?: WorkflowDefinition;
  onExecute?: (task: Task) => Promise<TaskResult>;
  onRollback?: (task: Task) => Promise<void>;
}
```

Key fields:
- **dependencies**: Array of task IDs that must complete before this task can start. The workflow engine resolves the dependency graph before scheduling.
- **workflow**: Enables nested workflows — a task can contain an entire workflow definition, creating hierarchical execution.
- **onExecute**: Callback invoked when the task is dispatched. Decouples task definition from execution logic.
- **onRollback**: Called when `rollbackOnFailure` is enabled and the task or a downstream task fails. Enables transactional semantics.

### TaskResult

```typescript
interface TaskResult {
  taskId: string;
  status: 'completed' | 'failed';
  result?: unknown;
  error?: string;
  duration?: number;
  agentId?: string;
}
```

Returned after task execution. The `duration` field (in milliseconds) feeds into performance metrics. The `agentId` records which agent executed the task for audit purposes.

### TaskAssignment

```typescript
interface TaskAssignment {
  taskId: string;
  agentId: string;
  assignedAt: Date;
  priority: TaskPriority;
}
```

Records the binding between a task and its assigned agent. Used by the coordination layer to track active assignments and prevent double-assignment.

---

## Memory Types

### MemoryType

```typescript
type MemoryType = 'task' | 'context' | 'event' | 'task-start' | 'task-complete' | 'workflow-state' | string;
```

Categorizes stored memories. The system automatically stores `task-start`, `task-complete`, and `workflow-state` entries. Custom types can be used for application-specific memory.

### Memory

```typescript
interface Memory {
  id: string;
  agentId: string;
  content: string;
  type: MemoryType;
  timestamp: Date;
  embedding?: number[];
  metadata?: Record<string, unknown>;
}
```

A single memory entry. The optional `embedding` field stores a vector representation for semantic search (used by the AgentDB backend with HNSW indexing). Metadata supports arbitrary key-value storage for filtering.

### MemoryQuery

```typescript
interface MemoryQuery {
  agentId?: string;
  type?: MemoryType;
  timeRange?: { start: Date; end: Date };
  metadata?: Record<string, unknown>;
  limit?: number;
  offset?: number;
}
```

Filter criteria for memory retrieval. All fields are optional — an empty query returns all memories (subject to `limit`). Fields are combined with AND logic.

### MemorySearchResult

```typescript
interface MemorySearchResult extends Memory {
  similarity?: number;
}
```

Extends `Memory` with an optional `similarity` score (0-1) returned by vector search operations. Higher values indicate closer semantic match.

---

## Workflow Types

### WorkflowStatus

```typescript
type WorkflowStatus = 'pending' | 'in-progress' | 'paused' | 'completed' | 'failed' | 'cancelled';
```

Extends `TaskStatus` with `paused` — workflows can be suspended and resumed, unlike individual tasks.

### WorkflowDefinition

```typescript
interface WorkflowDefinition {
  id: string;
  name: string;
  tasks: Task[];
  debug?: boolean;
  rollbackOnFailure?: boolean;
}
```

Declarative workflow specification:
- **tasks**: Ordered list of tasks. Dependencies between tasks are expressed via `task.dependencies`.
- **debug**: When `true`, the engine captures detailed execution traces in `WorkflowDebugInfo`.
- **rollbackOnFailure**: When `true`, completed tasks are rolled back (via `onRollback`) if a subsequent task fails.

### WorkflowState

```typescript
interface WorkflowState {
  id: string;
  status: WorkflowStatus;
  completedTasks: string[];
  currentTask?: string;
  startedAt: Date;
  completedAt?: Date;
}
```

Runtime state maintained by the workflow engine. Updated after each task completion. Persisted to memory for crash recovery.

### WorkflowResult

```typescript
interface WorkflowResult {
  id: string;
  status: WorkflowStatus;
  tasksCompleted: number;
  tasksFailed?: number;
  errors: string[];
  executionOrder?: string[];
  duration?: number;
}
```

Summary returned when a workflow finishes. The `executionOrder` field records the actual order tasks were executed (which may differ from definition order due to parallelism and dependency resolution).

### WorkflowMetrics

```typescript
interface WorkflowMetrics {
  totalTasks: number;
  completedTasks: number;
  failedTasks: number;
  averageExecutionTime: number;
  successRate: number;
}
```

Aggregated statistics across workflow execution. Used by the performance monitoring system and surfaced through the `status` CLI command.

### WorkflowDebugInfo

```typescript
interface WorkflowDebugInfo {
  executionTrace: Array<{ taskId: string; action: string; timestamp: Date }>;
  taskTimings: Map<string, { start: Date; end: Date; duration: number }>;
  memorySnapshots: Memory[];
  eventLog: string[];
}
```

Detailed debugging information captured when `debug: true`. Includes:
- **executionTrace**: Chronological record of all task state transitions
- **taskTimings**: Per-task timing for identifying bottlenecks
- **memorySnapshots**: Memory entries created during execution
- **eventLog**: Human-readable log of significant events

---

## Swarm & Coordination Types

### SwarmTopology

```typescript
type SwarmTopology = 'hierarchical' | 'mesh' | 'simple' | 'adaptive';
```

Determines how agents are organized:
- **hierarchical**: Tree structure with leaders delegating to workers. Best for anti-drift.
- **mesh**: Fully connected peers. Best for collaborative tasks.
- **simple**: Flat pool with no coordination overhead.
- **adaptive**: Dynamically switches topology based on load and task characteristics.

### SwarmConfig

```typescript
interface SwarmConfig {
  topology: SwarmTopology;
  maxAgents: number;
  strategy: string;
  // Additional topology-specific configuration
}
```

### SwarmState

Runtime state of the swarm including active agents, current topology, and coordination metadata.

### SwarmHierarchy

Tree structure representing parent-child relationships between agents in `hierarchical` topology.

### MeshConnection

Peer-to-peer connection record in `mesh` topology. Tracks connection health and message latency.

### AgentMessage

```typescript
interface AgentMessage {
  from: string;
  to: string;
  type: string;
  payload: unknown;
  timestamp?: Date;
}
```

Inter-agent communication primitive. The `type` field determines how the receiving agent processes the message. Used by both hierarchical delegation and mesh gossip protocols.

### AgentMetrics

```typescript
interface AgentMetrics {
  agentId: string;
  tasksCompleted: number;
  tasksFailed?: number;
  averageExecutionTime: number;
  successRate: number;
  health: 'healthy' | 'degraded' | 'unhealthy';
}
```

Per-agent performance tracking. The `health` field is derived from recent success rates and response times. Used by the adaptive topology to rebalance workloads.

### ConsensusDecision & ConsensusResult

Types supporting the Hive-Mind consensus system. `ConsensusDecision` represents a proposed decision, while `ConsensusResult` contains the outcome (accepted/rejected) and voting details.

---

## Plugin Types

### Plugin Interface

```typescript
interface Plugin {
  name: string;
  version: string;
  description?: string;
  initialize?: () => Promise<void>;
  destroy?: () => Promise<void>;
  extensions?: ExtensionPoint[];
}
```

Full plugin contract with lifecycle methods:
- **initialize**: Called when the plugin is loaded. Set up resources, register hooks.
- **destroy**: Called on shutdown. Clean up resources.
- **extensions**: Named extension points the plugin provides.

### ExtensionPoint

```typescript
interface ExtensionPoint {
  name: string;
  handler: (...args: unknown[]) => unknown;
  priority?: number;
}
```

A named hook point where plugins can inject behavior. The `priority` field determines execution order when multiple plugins extend the same point (lower = earlier).

### PluginMetadata & PluginManagerInterface

`PluginMetadata` stores registry information (author, license, dependencies). `PluginManagerInterface` defines the contract for plugin lifecycle management (load, unload, list, get).

---

## MCP Types

### MCPServerOptions

```typescript
interface MCPServerOptions {
  port?: number;
  host?: string;
  transport?: 'stdio' | 'http';
}
```

Configuration for starting the MCP server.

### MCPTool

```typescript
interface MCPTool {
  name: string;
  description: string;
  inputSchema: Record<string, unknown>;
  handler: (input: unknown) => Promise<MCPToolResult>;
}
```

A tool exposed through the MCP protocol. The `inputSchema` follows JSON Schema format for validation.

### MCPToolProvider, MCPToolResult

`MCPToolProvider` registers collections of tools. `MCPToolResult` wraps tool execution output with status and error information.

### MCPRequest & MCPResponse

Standard MCP protocol request/response envelopes following the Model Context Protocol specification.

---

## Backend Interfaces

### MemoryBackend

```typescript
interface MemoryBackend {
  store(memory: Memory): Promise<void>;
  retrieve(query: MemoryQuery): Promise<Memory[]>;
  update(id: string, memory: Partial<Memory>): Promise<void>;
  delete(id: string): Promise<void>;
  vectorSearch(embedding: number[], limit?: number): Promise<MemorySearchResult[]>;
  clearAgent(agentId: string): Promise<void>;
}
```

Abstract interface for memory storage. Implementations:
- **SQLiteBackend**: File-based relational storage
- **AgentDBBackend**: Vector-enabled storage with HNSW indexing (150x–12,500x faster search)
- **HybridBackend**: Combines both (default)

### SQLiteOptions

```typescript
interface SQLiteOptions {
  dbPath: string;
  timeout?: number;
}
```

### AgentDBOptions

```typescript
interface AgentDBOptions {
  dbPath: string;
  dimensions?: number;
  hnswM?: number;
  efConstruction?: number;
}
```

HNSW index parameters:
- **dimensions**: Vector dimensionality (default: 384 for MiniLM embeddings)
- **hnswM**: Number of connections per layer (higher = better recall, more memory)
- **efConstruction**: Build-time search width (higher = better index quality, slower build)

---

## Event Types

### AgentEvent

```typescript
interface AgentEvent {
  type: 'agent-spawned' | 'agent-terminated' | 'agent-error' | 'agent-status-change';
  agentId: string;
  timestamp: Date;
  data?: unknown;
}
```

### TaskEvent

```typescript
interface TaskEvent {
  type: 'task-created' | 'task-assigned' | 'task-started' | 'task-completed' | 'task-failed';
  taskId: string;
  timestamp: Date;
  data?: unknown;
}
```

### WorkflowEvent

```typescript
interface WorkflowEvent {
  type: 'workflow-started' | 'workflow-completed' | 'workflow-failed' | 'workflow-paused';
  workflowId: string;
  timestamp: Date;
  data?: unknown;
}
```

### PluginEvent

```typescript
interface PluginEvent {
  type: 'plugin-loaded' | 'plugin-unloaded' | 'plugin-error';
  pluginName: string;
  timestamp: Date;
  data?: unknown;
}
```

All event types follow a consistent structure with `type`, identifier, `timestamp`, and optional `data`. This uniformity enables generic event handling and logging.

---

## Error Hierarchy

All custom errors extend `V3Error`, which adds structured error codes and details to the standard `Error` class:

```typescript
class V3Error extends Error {
  code: string;
  details?: Record<string, unknown>;
}
```

### Specialized Error Classes

| Error Class | Code | Use Case |
|---|---|---|
| `ValidationError` | `VALIDATION_ERROR` | Invalid configuration, bad input, schema violations |
| `ExecutionError` | `EXECUTION_ERROR` | Task execution failures, runtime errors |
| `CoordinationError` | `COORDINATION_ERROR` | Agent communication failures, consensus failures |
| `PluginError` | `PLUGIN_ERROR` | Plugin initialization/execution failures |
| `MemoryError` | `MEMORY_ERROR` | Storage failures, query errors, backend unavailable |

### Usage Example

```typescript
try {
  await coordinator.assignTask(task, agentId);
} catch (error) {
  if (error instanceof CoordinationError) {
    // Handle coordination-specific failure
    console.error(`Coordination failed: ${error.code}`, error.details);
  }
}
```

The `code` field enables programmatic error handling without string matching on messages. The `details` object carries structured context about the failure.

---

## Design Decisions

### String Literal Unions with Extension (`| string`)
Types like `AgentType` and `TaskType` use `'known-value' | string` patterns. This provides autocomplete and documentation for built-in values while allowing custom values without type assertion. Trade-off: slightly weaker type safety in exchange for extensibility.

### Callback-based Task Execution
Tasks carry their own `onExecute` and `onRollback` callbacks rather than relying on a central dispatcher. This decouples task definition from execution strategy and enables tasks to be self-contained units of work.

### Single-file Type Definitions
All shared types live in one file (`v3/src/shared/types/index.ts`). This creates a single source of truth and avoids circular dependency issues that arise from splitting types across modules. The trade-off is a larger file, but types are organized into clear sections.

### Optional Fields
Most interface fields use `?` optional markers. This minimizes required configuration — users only specify what they need, and the system provides defaults. This follows the principle of progressive disclosure.

### Error Hierarchy with Codes
Custom error classes carry machine-readable `code` strings rather than relying on `instanceof` checks alone. This enables error handling across serialization boundaries (e.g., IPC, network) where prototype chains are lost.
