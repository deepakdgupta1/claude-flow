# Task Execution Module

## Overview

The Task Execution module provides the domain entity for tasks and the workflow orchestration engine that drives complex, multi-step execution pipelines. It handles dependency resolution, parallel execution, rollback, pause/resume, distributed workflows, and plugin extension points.

**Locations**:
- `v3/src/task-execution/domain/Task.ts` — Task domain entity
- `v3/src/task-execution/application/WorkflowEngine.ts` — Workflow orchestration service

---

## Task Domain Entity

**Location**: `v3/src/task-execution/domain/Task.ts`

The `Task` class represents a unit of work that can be assigned to an agent for execution.

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `id` | `string` | Unique task identifier |
| `type` | `TaskType` | One of `'code'`, `'test'`, `'review'`, `'design'`, `'deploy'`, `'workflow'`, or a custom string |
| `description` | `string` | Human-readable description of the task |
| `priority` | `TaskPriority` | One of `'high'`, `'medium'`, `'low'` |
| `status` | `TaskStatus` | One of `'pending'`, `'in-progress'`, `'completed'`, `'failed'`, `'cancelled'` |
| `assignedTo` | `string \| undefined` | Optional agent ID this task is assigned to |
| `dependencies` | `string[]` | Array of task IDs that must complete before this task can start |
| `metadata` | `object` | Arbitrary key-value metadata |
| `workflow` | `WorkflowDefinition \| undefined` | Optional nested workflow definition for composite tasks |
| `onExecute` | `() => Promise<any>` | Callback containing the actual task logic |
| `onRollback` | `() => Promise<void>` | Callback invoked on failure to undo side effects |

### Lifecycle Methods

#### `start(): void`
Transitions the task from `'pending'` to `'in-progress'` and records the start timestamp.

#### `complete(): void`
Transitions the task to `'completed'` and records the completion timestamp.

#### `fail(error?: string): void`
Transitions the task to `'failed'`, records the failure timestamp, and optionally stores the error message in metadata.

#### `cancel(): void`
Transitions the task to `'cancelled'`.

#### `getDuration(): number`
Returns the elapsed time in milliseconds between start and completion (or current time if still running).

### Query Methods

#### `areDependenciesResolved(completedTasks: Set<string>): boolean`

Checks whether all task IDs in the `dependencies` array are present in the provided `completedTasks` set. Returns `true` if the task has no dependencies or if all dependencies have been completed.

#### `isWorkflow(): boolean`

Returns `true` if the task's `type` is `'workflow'` and it has a `workflow` definition attached. Workflow tasks support nested composition — a single task can contain an entire sub-workflow.

#### `getPriorityValue(): number`

Maps the priority string to a numeric value for sorting:

| Priority | Value |
|----------|-------|
| `high` | 3 |
| `medium` | 2 |
| `low` | 1 |

### Static Methods

#### `static sortByPriority(tasks: Task[]): Task[]`

Sorts an array of tasks from highest to lowest priority using `getPriorityValue()`.

#### `static resolveExecutionOrder(tasks: Task[]): Task[]`

Performs a **topological sort** on the task array based on their `dependencies`. This ensures tasks execute in a valid order where all dependencies are satisfied before a task begins.

Key behaviors:
- **Circular dependency detection**: Throws an error if a cycle is found in the dependency graph.
- **Priority-aware**: Within each dependency tier (tasks at the same depth in the graph), tasks are sorted by priority (high → low).
- Returns a flat array representing the correct execution sequence.

---

## WorkflowEngine

**Location**: `v3/src/task-execution/application/WorkflowEngine.ts`

The `WorkflowEngine` is the orchestration service that drives complex multi-step workflows, coordinating task execution across agents with full lifecycle management.

### Constructor

```typescript
constructor(
  coordinator: SwarmCoordinator,
  memoryBackend?: IMemoryBackend,
  eventBus?: EventBus,
  pluginManager?: PluginManager
)
```

| Parameter | Required | Description |
|-----------|----------|-------------|
| `coordinator` | Yes | The `SwarmCoordinator` instance managing the agent pool |
| `memoryBackend` | No | Persistence layer for storing execution events and state |
| `eventBus` | No | Event bus for publishing workflow lifecycle events |
| `pluginManager` | No | Plugin manager for pre/post workflow extension points |

### Core Methods

#### `executeTask(task: Task, agentId: string): Promise<TaskResult>`

Executes a single task on a specific agent via the coordinator.

**Flow**:
1. Delegates execution to `coordinator.executeTask(agentId, task)`.
2. Stores a `task:started` event in the memory backend (if available).
3. On completion, stores a `task:completed` event.
4. Returns the `TaskResult` from the agent.

#### `executeWorkflow(workflow: WorkflowDefinition): Promise<WorkflowResult>`

The main orchestration method for executing a complete workflow definition.

**Execution Flow**:

1. **Initialize state tracking**: Creates execution state including timings, an event log, and memory snapshots for debugging and metrics.

2. **Plugin pre-hook**: Invokes the `workflow.beforeExecute` extension point via the plugin manager, allowing plugins to modify the workflow or inject behavior before execution begins.

3. **Run workflow** (`runWorkflow()`):
   - Performs topological sort on all tasks to resolve dependency order.
   - Executes tasks sequentially in dependency order.
   - Supports **pause/cancel**: The execution loop checks for pause/cancel flags on each iteration. When paused, it enters a polling loop until resumed or cancelled.
   - Supports **nested workflows**: If a task is itself a workflow (`task.isWorkflow()` returns `true`), `executeWorkflow` is called recursively on the nested workflow definition.
   - **Automatic agent selection**: If a task has no `assignedTo` value, the engine selects a suitable agent from the coordinator's pool based on capability matching.

4. **Rollback on failure**: If any task fails, the engine executes rollback in **reverse order** — iterating through completed tasks from last to first, calling each task's `onRollback` callback. This ensures side effects are undone in the correct order.

5. **Plugin post-hook**: Invokes the `workflow.afterExecute` extension point, allowing plugins to react to the workflow result, collect metrics, or trigger downstream effects.

#### `startWorkflow(): void`

Non-blocking variant of `executeWorkflow`. Kicks off the workflow in the background without awaiting completion. Useful for fire-and-forget scenarios or when monitoring progress via events.

#### `pauseWorkflow(): void`

Pauses a running workflow. The execution loop in `runWorkflow()` detects the pause flag and enters a polling state, holding the current position in the task sequence. No new tasks are started while paused.

#### `resumeWorkflow(): void`

Resumes a paused workflow. Clears the pause flag, causing the polling loop to exit and task execution to continue from where it left off.

### Parallel and Distributed Execution

#### `executeParallel(tasks: Task[]): Promise<TaskResult[]>`

Delegates to the coordinator's `executeTasksConcurrently()` method, running all provided tasks in parallel across available agents.

#### `executeDistributedWorkflow(workflow: WorkflowDefinition, coordinators: SwarmCoordinator[]): Promise<WorkflowResult>`

Distributes tasks across multiple `SwarmCoordinator` instances for horizontal scaling.

**Strategy**: Chunk-based distribution — tasks are divided into roughly equal chunks, with each chunk assigned to a different coordinator. This enables multi-coordinator scaling where different agent pools handle different portions of the workload.

### Observability

#### `getWorkflowMetrics(): WorkflowMetrics`

Returns aggregate metrics for the workflow:
- Total tasks executed
- Total tasks completed / failed
- Average execution time per task
- Overall success rate (completed / total)

#### `getWorkflowDebugInfo(): WorkflowDebugInfo`

Returns detailed debugging information:
- **Execution trace**: Ordered list of task executions with agent assignments
- **Task timings**: Start time, end time, and duration for each task
- **Memory snapshots**: State snapshots captured during execution
- **Event log**: Chronological log of all workflow events

#### `restoreWorkflow(workflowId: string): Promise<WorkflowState>`

Loads persisted workflow state from the memory backend, enabling workflow recovery after crashes or restarts. The restored state includes the task execution position, completed task set, and any intermediate results.

---

## Design Decisions

### Topological Sort for Dependency Resolution

`Task.resolveExecutionOrder()` uses topological sorting to guarantee correct execution ordering. This is essential for workflows where task B depends on the output of task A. The algorithm also detects circular dependencies at sort time, failing fast rather than deadlocking at runtime.

### Plugin Extension Points

The `workflow.beforeExecute` and `workflow.afterExecute` hooks provide a clean extension mechanism. Plugins can:
- Validate or modify workflows before execution
- Inject additional tasks or checks
- Collect custom metrics after execution
- Trigger notifications or downstream workflows

### Distributed Workflows

The `executeDistributedWorkflow` method enables scaling beyond a single coordinator's agent pool. By distributing task chunks across multiple coordinators, the system can leverage larger agent pools or geographically distributed resources.

### Complete Audit Trail

Every task start, completion, and failure is logged to the memory backend and event log. Combined with `getWorkflowDebugInfo()`, this provides a complete audit trail for debugging, compliance, and performance analysis.

### Rollback in Reverse Order

When a task fails, rollback proceeds from the most recently completed task backward. This mirrors the undo pattern seen in database transactions and ensures that dependent side effects are undone before their prerequisites.

---

## Usage Example

```typescript
import { Task } from './domain/Task';
import { WorkflowEngine } from './application/WorkflowEngine';

// Define tasks with dependencies
const compile = new Task({
  id: 'compile',
  type: 'code',
  priority: 'high',
  description: 'Compile the project',
  onExecute: async () => { /* compile logic */ },
  onRollback: async () => { /* cleanup artifacts */ },
});

const test = new Task({
  id: 'test',
  type: 'test',
  priority: 'high',
  description: 'Run test suite',
  dependencies: ['compile'],
  onExecute: async () => { /* test logic */ },
  onRollback: async () => { /* cleanup test state */ },
});

const deploy = new Task({
  id: 'deploy',
  type: 'deploy',
  priority: 'medium',
  description: 'Deploy to staging',
  dependencies: ['test'],
  onExecute: async () => { /* deploy logic */ },
  onRollback: async () => { /* rollback deployment */ },
});

// Resolve execution order (topological sort)
const ordered = Task.resolveExecutionOrder([deploy, compile, test]);
// Result: [compile, test, deploy]

// Execute via WorkflowEngine
const engine = new WorkflowEngine(coordinator, memoryBackend, eventBus);

const result = await engine.executeWorkflow({
  id: 'build-and-deploy',
  tasks: [compile, test, deploy],
});

// Inspect metrics
const metrics = engine.getWorkflowMetrics();
// { total: 3, completed: 3, failed: 0, avgDuration: 150, successRate: 1.0 }
```
