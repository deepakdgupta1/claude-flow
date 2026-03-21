# Neural Learning

## Overview

The Neural Learning module provides a complete self-optimizing learning system that enables agents to learn from experience, recognize patterns, and improve over time. It integrates reinforcement learning, trajectory tracking, knowledge distillation, and consolidation into a unified pipeline.

**Location**: `v3/@claude-flow/neural/`

**Performance Targets**:

| Metric | Target |
|--------|--------|
| SONA adaptation | < 0.05ms |
| Pattern matching | < 1ms |
| Learning step | < 10ms |

---

## NeuralLearningSystem (Facade)

The `NeuralLearningSystem` class is the primary entry point. It integrates `SONAManager`, `ReasoningBank`, and `PatternLearner` behind a unified API.

### Lifecycle

```typescript
const neural = new NeuralLearningSystem(config);
await neural.initialize();

// Begin a task trajectory
const trajectoryId = await neural.beginTask(context, 'code');

// Record learning steps during execution
await neural.recordStep(trajectoryId, action, reward, stateEmbedding);

// Complete the task — triggers quality judgment, distillation, and pattern extraction
await neural.completeTask(trajectoryId, qualityScore);
```

### Methods

| Method | Description |
|--------|-------------|
| `initialize()` | Initializes the SONA subsystem and all internal components |
| `beginTask(context, domain)` | Starts trajectory tracking. Returns a `trajectoryId` |
| `recordStep(trajectoryId, action, reward, stateEmbedding)` | Records a single learning step within a trajectory |
| `completeTask(trajectoryId, quality)` | Completes the trajectory, stores it in the ReasoningBank, judges quality, distills memory, and extracts patterns |
| `findPatterns(queryEmbedding, k)` | Finds the `k` most similar learned patterns via vector similarity |
| `retrieveMemories(queryEmbedding, k)` | Retrieves the `k` most relevant memories from the ReasoningBank |
| `triggerLearning()` | Manually triggers a learning cycle followed by consolidation |
| `getStats()` | Returns combined statistics from all subsystems |

### Domains

The `domain` parameter in `beginTask()` configures learning behavior:

- `'code'` — Code generation, refactoring, and debugging
- `'creative'` — Creative writing and ideation
- `'reasoning'` — Logical reasoning and problem-solving
- `'chat'` — Conversational interaction
- `'math'` — Mathematical computation
- `'general'` — Default domain

---

## SONA Manager (Self-Optimizing Neural Architecture)

SONA provides adaptive learning that dynamically adjusts its behavior based on the current context and resource constraints.

### Learning Modes

| Mode | Latency | Description |
|------|---------|-------------|
| `RealTime` | < 0.05ms | Minimal overhead, instant adaptation. For latency-critical paths |
| `Balanced` | ~1ms | Default tradeoff between learning quality and speed |
| `Research` | ~10ms | Maximum learning quality, higher computational overhead |
| `Edge` | < 0.1ms | Resource-constrained environments (low memory, limited CPU) |
| `Batch` | Variable | Offline batch learning over accumulated trajectories |

Mode switching happens dynamically based on system load, task type, and explicit configuration. Each mode adjusts:

- Learning rate
- Exploration vs. exploitation balance
- Memory consolidation frequency
- Pattern extraction depth

### Trajectory Tracking

SONA tracks complete task trajectories:

1. **Begin** — Initialize trajectory with context and domain
2. **Record Steps** — Each action, its reward, and state embedding are recorded
3. **Complete** — Trajectory is finalized with a quality score and forwarded to the ReasoningBank

---

## ReasoningBank

The ReasoningBank implements a 4-step intelligence pipeline that mirrors how humans learn from experience:

```
┌──────────┐    ┌─────────┐    ┌──────────┐    ┌──────────────┐
│ RETRIEVE │───►│  JUDGE  │───►│ DISTILL  │───►│ CONSOLIDATE  │
│          │    │         │    │          │    │              │
│ HNSW     │    │ Success/│    │ LoRA     │    │ EWC++        │
│ vector   │    │ Failure │    │ extract  │    │ prevent      │
│ search   │    │ verdict │    │ learnings│    │ forgetting   │
└──────────┘    └─────────┘    └──────────┘    └──────────────┘
```

### Pipeline Steps

**1. RETRIEVE** — Fetch relevant patterns via HNSW vector search. When a new trajectory arrives, the ReasoningBank searches for similar past experiences to provide context for judgment.

**2. JUDGE** — Evaluate trajectory quality with verdicts. Each trajectory receives a `success` or `failure` verdict based on:
- Task completion quality score
- Comparison with similar past trajectories
- Reward signal analysis

**3. DISTILL** — Extract key learnings via LoRA (Low-Rank Adaptation) distillation. Successful trajectories are compressed into `DistilledMemory` objects that capture the essential patterns without the full trajectory overhead. This enables lightweight fine-tuning without full model retraining.

**4. CONSOLIDATE** — Prevent catastrophic forgetting via EWC++ (Elastic Weight Consolidation). When new patterns are learned, EWC++ ensures that previously learned knowledge is preserved by constraining updates to parameters that are important for past tasks.

### Storage

The ReasoningBank stores:
- Raw trajectories with their verdicts
- Distilled memories extracted from successful trajectories
- Consolidation checkpoints for rollback

---

## Pattern Learner

The PatternLearner extracts reusable patterns from successful trajectories.

### Methods

| Method | Description |
|--------|-------------|
| `extractPatterns(trajectory)` | Analyzes a successful trajectory and extracts reusable patterns |
| `findMatches(queryEmbedding, k)` | Vector similarity search for the `k` most relevant patterns |
| `getPatternStats()` | Returns statistics on pattern library size, usage frequency, and evolution |

### Pattern Evolution

Patterns are not static. The PatternLearner tracks:
- How often each pattern is applied
- Success rate when the pattern is used
- Drift in the pattern's embedding over time
- Confidence score that increases with successful applications

---

## RL Algorithms

The module provides 9 reinforcement learning algorithm implementations, selectable via factory:

| Algorithm | Type | Description |
|-----------|------|-------------|
| **Q-Learning** | Tabular | Classic tabular RL with state-action value function |
| **SARSA** | Tabular | On-policy temporal difference learning |
| **A2C** | Policy Gradient | Advantage Actor-Critic with value baseline |
| **PPO** | Policy Gradient | Proximal Policy Optimization with clipped objective |
| **DQN** | Deep RL | Deep Q-Network with experience replay |
| **Decision Transformer** | Sequence | Sequence modeling approach to RL (transformer-based) |
| **Curiosity Module** | Exploration | Intrinsic motivation for exploration of novel states |

### Factory

```typescript
import { createAlgorithm, getDefaultConfig } from '@claude-flow/neural/rl';

const config = getDefaultConfig('ppo');
const algorithm = createAlgorithm('ppo', config);
```

Each algorithm exposes:
- `selectAction(state)` — Choose action given current state
- `update(transition)` — Update policy from observed transition
- `getPolicy()` — Export current policy parameters

---

## SONA Integration (@ruvector/sona)

The `SONALearningEngine` bridges the TypeScript neural system with the native Rust SONA implementation for maximum performance.

| Feature | Description |
|---------|-------------|
| Context-based adaptation | SONA adapts learning parameters based on the current task context |
| Pattern storage | Learned patterns are persisted via the Rust backend |
| Pattern retrieval | Sub-millisecond pattern lookup via native HNSW |

---

## Learning Modes (Detail)

All modes extend `BaseModeImplementation`, which provides:
- Common trajectory handling
- Metric collection
- Mode transition hooks

### Mode Implementations

**RealTimeMode**: Strips learning to the absolute minimum. Only records trajectory outcomes; pattern extraction is deferred. Suitable for production inference paths where latency matters.

**BalancedMode**: The default mode. Performs inline pattern extraction and periodic consolidation. Good for most development and runtime scenarios.

**ResearchMode**: Enables full trajectory analysis, cross-trajectory pattern correlation, and deep consolidation. Use for offline analysis or when learning quality is paramount.

**EdgeMode**: Designed for resource-constrained environments. Limits memory usage, reduces embedding dimensions, and batches consolidation. Suitable for edge devices or memory-limited containers.

**BatchMode**: Accumulates trajectories without inline learning. Learning is triggered explicitly via `triggerLearning()` or on a schedule. Ideal for offline training pipelines.

---

## Design Decisions

| Decision | Rationale |
|----------|-----------|
| RETRIEVE→JUDGE→DISTILL→CONSOLIDATE pipeline | Mirrors human experiential learning: recall similar experiences, evaluate outcomes, extract lessons, integrate without forgetting |
| EWC++ for consolidation | Prevents catastrophic forgetting when learning new patterns, a well-known problem in continual learning |
| Multiple RL algorithms | Different task types benefit from different optimization strategies; factory pattern makes selection easy |
| Mode system | Deployment contexts vary widely (edge vs. cloud, real-time vs. batch); mode system adapts learning without code changes |
| LoRA distillation | Lightweight fine-tuning that captures essential patterns without the overhead of full trajectory storage |
| Domain-specific configuration | Code generation, reasoning, and creative tasks have fundamentally different reward structures and pattern shapes |
