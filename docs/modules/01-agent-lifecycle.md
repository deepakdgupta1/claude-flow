# Agent Lifecycle Module

## Overview

The Agent Lifecycle module defines the `Agent` domain entity — the fundamental building block representing an individual AI agent in the Claude Flow system. Every agent spawned, coordinated, or terminated flows through this module.

**Location**: `v3/src/agent-lifecycle/domain/Agent.ts`

---

## Agent Domain Entity

The `Agent` class implements the `IAgent` interface from shared types and encapsulates all state and behavior for a single agent instance.

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `id` | `string` | Unique identifier for the agent |
| `type` | `AgentType` | One of `'coder'`, `'tester'`, `'reviewer'`, `'coordinator'`, `'designer'`, `'deployer'`, or a custom string |
| `status` | `AgentStatus` | One of `'active'`, `'idle'`, `'busy'`, `'terminated'`, `'error'` |
| `capabilities` | `string[]` | List of capability strings the agent possesses (e.g., `'code'`, `'test'`, `'review'`) |
| `role` | `AgentRole` | One of `'leader'`, `'worker'`, `'peer'` |
| `parent` | `string \| undefined` | Optional ID linking to a parent agent in hierarchical topologies |
| `metadata` | `object` | Arbitrary key-value metadata attached to the agent |
| `createdAt` | `number` | Timestamp (ms) when the agent was created |
| `lastActive` | `number` | Timestamp (ms) of the agent's last activity |

### Constructor

```typescript
constructor(config: AgentConfig)
```

`AgentConfig` accepts:
- `id` (required) — unique agent identifier
- `type` (required) — agent type
- `capabilities` (optional) — array of capability strings
- `role` (optional) — agent role, defaults vary by context
- `parent` (optional) — parent agent ID
- `metadata` (optional) — arbitrary metadata

On construction:
- `status` is initialized to `'active'`
- `createdAt` and `lastActive` are both set to `Date.now()`

---

## Key Methods

### `executeTask(task: Task): Promise<TaskResult>`

The core method for task execution. This is the primary entry point for running work on an agent.

**Flow**:
1. Checks agent availability — the agent must be in `'active'` or `'idle'` status. If the agent is `'terminated'`, `'error'`, or `'busy'`, the call will throw or return a failed result.
2. Sets the agent's status to `'busy'`.
3. Calls the task's `onExecute` callback — the actual business logic lives in the task, not the agent.
4. Calls `processTaskExecution(task)` for scheduling overhead simulation.
5. Returns a `TaskResult` containing:
   - `status`: `'completed'` or `'failed'`
   - `duration`: execution time in milliseconds
   - `agentId`: the executing agent's ID
6. On error, catches the exception and returns a `'failed'` result with the error details.

### `processTaskExecution(task): Promise<void>` (private)

Adds a minimal overhead delay based on task priority to simulate real-world scheduling differences:

| Priority | Delay |
|----------|-------|
| `high` | 1ms |
| `medium` | 5ms |
| `low` | 10ms |

The actual work is performed by the task's `onExecute` callback — this method only adds scheduling-level overhead.

### `hasCapability(capability: string): boolean`

Returns `true` if the agent's `capabilities` array includes the given capability string.

### `canExecute(taskType: string): boolean`

Maps task types to required capabilities and checks if the agent possesses them:

| Task Type | Required Capability |
|-----------|-------------------|
| `code` | `code` |
| `test` | `test` |
| `review` | `review` |
| `design` | `design` |
| `deploy` | `deploy` |
| `refactor` | `refactor` |
| `debug` | `debug` |

Returns `true` if the agent has the capability needed for the given task type.

### State Transitions

All state transition methods include guard clauses to prevent invalid transitions.

- **`terminate()`**: Sets status to `'terminated'`. Guard: cannot terminate an already-terminated agent.
- **`setIdle()`**: Sets status to `'idle'`. Guard: agent must be in a non-terminal state.
- **`activate()`**: Sets status to `'active'`. Guard: agent must not be terminated or in error.

### Serialization and Factory

- **`toJSON(): IAgent`**: Serializes the agent to a plain `IAgent` object suitable for storage or transmission.
- **`static fromConfig(config: AgentConfig): Agent`**: Factory method that creates a new `Agent` instance from an `AgentConfig` object.

---

## Design Decisions

### Callback-Based Task Execution

The `Agent` class is intentionally a lightweight orchestration wrapper. It does not contain business logic — instead, the task's `onExecute` callback carries the actual work. This separation means:

- Agents are reusable across different task types without modification.
- Task logic can be defined and tested independently of the agent.
- The agent focuses purely on lifecycle management, availability checks, and metrics.

### Priority-Based Processing Overhead

The `processTaskExecution` method introduces artificial delays based on priority. This design choice simulates real-world scheduling behavior where high-priority tasks receive preferential treatment and lower-priority tasks experience greater scheduling latency.

### Status Guard Clauses

Every state transition and task execution checks the current status before proceeding. This prevents:

- Executing tasks on terminated or errored agents
- Invalid state transitions (e.g., activating a terminated agent)
- Concurrent execution conflicts (agents set to `'busy'` during execution)

### Domain Entity Isolation

The `Agent` class is a pure domain entity with no infrastructure dependencies. It does not import database clients, network libraries, or framework-specific code. This ensures:

- The domain logic is testable in isolation
- Infrastructure concerns (persistence, messaging) are handled at higher layers
- The entity can be used across different runtime environments

---

## Relationship to Other Modules

### SwarmCoordinator

The `SwarmCoordinator` (see [03-swarm-coordination.md](./03-swarm-coordination.md)) manages pools of `Agent` instances. It handles:

- Spawning agents via `Agent.fromConfig()` or the constructor
- Tracking agent metrics (tasks completed, success rate, health)
- Load-balanced task distribution across available agents
- Topology-aware connections between agents

### Task Execution

Tasks (see [02-task-execution.md](./02-task-execution.md)) are assigned to agents through the coordination layer. The `Agent.executeTask()` method is the bridge between coordination and execution — it receives a `Task` object and drives it through the execution lifecycle.

### Memory Backends

Agent spawn and terminate events are tracked in memory backends. The agent itself does not write to memory — this responsibility falls to the `SwarmCoordinator` and `WorkflowEngine`, which store events alongside execution results and metrics.

---

## Usage Example

```typescript
import { Agent } from './domain/Agent';

// Create an agent
const agent = Agent.fromConfig({
  id: 'agent-001',
  type: 'coder',
  capabilities: ['code', 'refactor', 'debug'],
  role: 'worker',
  metadata: { language: 'typescript' },
});

// Check capabilities
agent.hasCapability('code');       // true
agent.canExecute('code');          // true
agent.canExecute('test');          // false

// Execute a task
const result = await agent.executeTask(task);
// result: { status: 'completed', duration: 42, agentId: 'agent-001' }

// Lifecycle transitions
agent.setIdle();      // status → 'idle'
agent.activate();     // status → 'active'
agent.terminate();    // status → 'terminated'
```
