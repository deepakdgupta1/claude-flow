# Module 07: Hooks System

**Location**: `v3/@claude-flow/hooks/`

**Purpose**: Event-driven lifecycle hooks with ReasoningBank learning integration. Provides the observability and learning feedback loop for the entire Claude Flow system.

---

## Overview

The Hooks System is the nervous system of Claude Flow. It intercepts lifecycle events across the platform — edits, commands, tasks, sessions, agent actions — and feeds them into a learning pipeline powered by the ReasoningBank. Background workers run continuously to analyze, optimize, and learn from system behavior without blocking the main execution flow.

---

## Core Components

### HookRegistry

Central registry for hook handlers. All hooks must be registered before they can be executed.

- **`registerHook(event, handler, options?)`** — Registers a handler for a specific event type. Options include `priority` (execution order) and `once` (single-fire).
- **`unregisterHook(event, handlerId)`** — Removes a previously registered handler.

Handlers are stored per-event and sorted by priority at registration time.

### HookExecutor

Executes all registered hooks for a given event type, passing context through the chain.

- **`executeHooks(event, context)`** — Runs all handlers registered for `event` in priority order. Returns a `HookExecutionResult` containing:
  - `success`: boolean
  - `results`: array of individual handler results
  - `duration`: total execution time in milliseconds
  - `errors`: array of any errors encountered (non-fatal by default)

Execution is sequential by default; handlers can be marked as parallel-safe for concurrent execution.

### ReasoningBank (Hooks-Specific)

Vector-based pattern learning system integrated into the hooks pipeline. Learns from hook executions to provide guidance for future operations.

**Core Types**:
- **`GuidancePattern`** — A learned pattern with vector embedding, confidence score, and associated context.
- **`GuidanceResult`** — Result of querying the ReasoningBank for guidance, containing matching patterns and recommended actions.
- **`RoutingResult`** — Result of task routing decisions, including recommended agent type, model tier, and confidence.

**Key Interface**:
- **`GuidanceProvider`** — Generates Claude-visible output from learned patterns. This is how the ReasoningBank's knowledge surfaces to the user and to Claude Code sessions.

### DaemonManager

Manages long-running background daemons that provide continuous system monitoring and learning.

**Built-in Daemons**:
- **MetricsDaemon** — Collects and aggregates system metrics (response times, error rates, resource usage).
- **SwarmMonitorDaemon** — Monitors active swarm health, detects drift, and triggers rebalancing.
- **HooksLearningDaemon** — Processes hook execution results through the ReasoningBank for pattern extraction.

### StatuslineGenerator

Generates dynamic statusline content for Claude Code integration. Provides real-time system state information including:
- Active agent count and status
- Current swarm topology
- Memory usage and health
- Background worker status
- Recent hook execution summaries

### OfficialHooksBridge

Maps V3 hook events to the official Claude Code hook event system, enabling seamless integration with Claude Code's native hook infrastructure.

**Mapped Events**:
- `pre-edit` → Claude Code pre-edit hook
- `post-edit` → Claude Code post-edit hook
- `pre-command` → Claude Code pre-command hook (PreToolUse)
- `post-command` → Claude Code post-command hook (PostToolUse)

### SwarmCommunication

Inter-agent communication layer built on the hooks system.

- **Pattern Broadcast** — Shares learned patterns across all agents in a swarm.
- **Consensus Requests** — Initiates consensus votes among agents for critical decisions.
- **Task Handoff** — Transfers task ownership between agents with full context preservation.

---

## Hook Events (27 Total)

### Core Events (6)

| Event | Trigger | Context |
|-------|---------|---------|
| `pre-edit` | Before file edit | `{file, oldContent, newContent}` |
| `post-edit` | After file edit | `{file, oldContent, newContent, success}` |
| `pre-command` | Before command execution | `{command, args, cwd}` |
| `post-command` | After command execution | `{command, args, exitCode, output}` |
| `pre-task` | Before task starts | `{taskId, description, agentType}` |
| `post-task` | After task completes | `{taskId, success, duration, metrics}` |

### Session Events (4)

| Event | Trigger | Context |
|-------|---------|---------|
| `session-start` | Session begins | `{sessionId, config, resumedFrom?}` |
| `session-end` | Session ends | `{sessionId, duration, metrics}` |
| `session-restore` | Session restored from persistence | `{sessionId, checkpoint}` |
| `notify` | Notification dispatch | `{level, message, metadata}` |

### Intelligence Events (7)

| Event | Trigger | Context |
|-------|---------|---------|
| `route` | Task routing decision needed | `{task, availableAgents, constraints}` |
| `route-task` | V2 compatibility routing | `{task, complexity}` |
| `explain` | Explanation requested | `{topic, depth, audience}` |
| `pretrain` | Model pretraining triggered | `{modelType, epochs, data}` |
| `build-agents` | Agent pool construction | `{agentTypes, count, config}` |
| `metrics` | Metrics collection point | `{category, values}` |
| `transfer` | Pattern transfer between projects | `{source, target, patterns}` |

### Learning Events (4 event types, multiple sub-events)

The `intelligence` event carries sub-events:

| Sub-Event | Purpose |
|-----------|---------|
| `trajectory-start` | Begin tracking a learning trajectory |
| `trajectory-step` | Record a step in the trajectory |
| `trajectory-end` | Complete trajectory and extract patterns |
| `pattern-store` | Store a learned pattern in ReasoningBank |
| `pattern-search` | Search for relevant patterns |
| `stats` | Learning system statistics |
| `attention` | Attention mechanism updates |

### Worker Events (4 event types, multiple sub-events)

The `worker` event carries sub-events:

| Sub-Event | Purpose |
|-----------|---------|
| `list` | List all registered workers |
| `dispatch` | Dispatch a worker for execution |
| `status` | Query worker status |
| `detect` | Auto-detect appropriate worker for a task |

### Coverage Events (3)

| Event | Trigger | Context |
|-------|---------|---------|
| `coverage-route` | Route coverage analysis task | `{files, threshold}` |
| `coverage-suggest` | Suggest tests for uncovered code | `{uncoveredPaths}` |
| `coverage-gaps` | Report coverage gaps | `{gaps, severity}` |

### Compatibility Events (4)

| Event | Maps To | Purpose |
|-------|---------|---------|
| `pre-bash` | `pre-command` | V2 alias for pre-command |
| `post-bash` | `post-command` | V2 alias for post-command |
| `statusline` | StatuslineGenerator | Request statusline update |
| `progress` | MetricsDaemon | Report progress update |

---

## Background Workers (12)

Workers run asynchronously via the DaemonManager without blocking the main execution flow.

| Worker | Priority | Description |
|--------|----------|-------------|
| `ultralearn` | normal | Deep knowledge acquisition from code patterns and documentation |
| `optimize` | **high** | Performance optimization recommendations based on observed patterns |
| `consolidate` | low | Memory consolidation — merges, deduplicates, and compresses stored patterns |
| `predict` | normal | Predictive preloading of likely-needed context and resources |
| `audit` | **critical** | Security analysis of code changes, dependency vulnerabilities |
| `map` | normal | Codebase mapping — maintains an up-to-date structural index |
| `preload` | low | Resource preloading based on predicted access patterns |
| `deepdive` | normal | Deep code analysis for complex refactoring suggestions |
| `document` | normal | Auto-documentation generation from code changes |
| `refactor` | normal | Refactoring opportunity detection and suggestion |
| `benchmark` | normal | Performance benchmarking of critical paths |
| `testgaps` | normal | Test coverage analysis and gap identification |

Workers are dispatched via the `worker` hook event or directly through MCP tools.

---

## MCP Tool Integration

### Core Hook MCP Tools

The hooks system exposes the following MCP tools for invocation from Claude Code sessions:

| Tool | Description |
|------|-------------|
| `preEditTool` | Trigger pre-edit hooks before a file modification |
| `postEditTool` | Trigger post-edit hooks after a file modification |
| `routeTaskTool` | Route a task to the appropriate agent/model |
| `metricsTool` | Collect and report system metrics |
| `preCommandTool` | Trigger pre-command hooks before command execution |
| `postCommandTool` | Trigger post-command hooks after command execution |
| `daemonStatusTool` | Query status of all background daemons |
| `statuslineTool` | Generate current statusline content |

### Worker MCP Tools

| Tool | Description |
|------|-------------|
| `workerRunTool` | Run a specific background worker |
| `workerStatusTool` | Query status of a specific worker |
| `workerAlertsTool` | Get alerts from worker execution |
| `workerHistoryTool` | View execution history for a worker |
| `workerStatuslineTool` | Get worker-specific statusline content |
| `workerRunAllTool` | Run all workers simultaneously |
| `workerStartTool` | Start a stopped worker |
| `workerStopTool` | Stop a running worker |

---

## Session Integration

### Lifecycle Methods

- **`onSessionStart()`** — Called when a Claude Code session begins. Initializes hook context, starts background daemons, restores any persisted state.
- **`onSessionEnd()`** — Called when a session ends. Flushes metrics, persists learned patterns, stops daemons gracefully.
- **`formatSessionStartOutput()`** — Generates the formatted output shown to users at session start, including system status and available capabilities.
- **`generateShellHook()`** — Generates shell hook scripts for integration with the user's terminal environment.

---

## Initialization

### `initializeHooks(options)`

Primary initialization function for the hooks system.

**Options**:
- `enableDaemons` (boolean, default: `true`) — Whether to start background daemons.
- `enableStatusline` (boolean, default: `true`) — Whether to enable statusline generation.

**Initialization Sequence**:
1. Create HookRegistry instance
2. Create HookExecutor with the registry
3. Initialize ReasoningBank
4. Register built-in hooks and bridge to Claude Code events
5. Start StatuslineGenerator (if enabled)
6. Start DaemonManager and background workers (if enabled)

### `runHook(event, context)`

Quick execution helper. Looks up the event in the registry and executes all handlers.

```typescript
await runHook('post-edit', { file: 'src/index.ts', success: true });
```

### `addHook(event, handler, options?)`

Simplified registration API for adding hooks without direct registry access.

```typescript
addHook('pre-task', async (ctx) => {
  console.log(`Starting task: ${ctx.description}`);
}, { priority: 10 });
```

---

## Design Decisions

1. **Event-driven architecture** — Hooks use an event-driven model to maintain loose coupling between system components. Any module can emit or listen for events without direct dependencies.

2. **ReasoningBank integration** — Every hook execution feeds into the ReasoningBank learning pipeline, enabling the system to learn from operational patterns and provide increasingly accurate guidance over time.

3. **Asynchronous background workers** — Workers run asynchronously without blocking the main execution flow. Critical workers (like `audit`) are prioritized, while low-priority workers (like `consolidate`) yield to more important work.

4. **V2 compatibility layer** — The `pre-bash`/`post-bash` aliases and `route-task` event ensure existing V2 configurations continue to work during migration to V3.

5. **MCP tool exposure** — Exposing hooks as MCP tools allows them to be invoked directly from Claude Code sessions, enabling users to trigger learning, routing, and analysis on demand.

6. **Priority-based execution** — Hooks execute in priority order, allowing critical handlers (security checks, validation) to run before informational handlers (logging, metrics).
