# Memory System

## Overview

The Memory System provides persistent, queryable storage for agent knowledge with both structured relational queries and semantic vector search. It is a core pillar of Claude Flow V3, enabling agents to store, retrieve, and reason over accumulated knowledge across sessions.

**Key Locations**:

| Path | Description |
|------|-------------|
| `v3/src/memory/domain/Memory.ts` | Memory domain entity |
| `v3/src/memory/infrastructure/` | Backend implementations (SQLiteBackend, AgentDBBackend, HybridBackend) |
| `v3/@claude-flow/memory/` | Full memory package |
| `v3/@claude-flow/embeddings/` | Embeddings package |

---

## Memory Domain Entity

The `MemoryEntity` class implements the `Memory` interface and serves as the core data structure for all stored knowledge.

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `id` | `string` | Unique identifier |
| `agentId` | `string` | Owning agent's identifier |
| `content` | `string` | The stored content |
| `type` | `MemoryType` | Classification of the memory |
| `timestamp` | `Date` | Creation time |
| `embedding` | `number[] \| undefined` | Optional vector embedding |
| `metadata` | `Record<string, unknown>` | Arbitrary metadata |

### MemoryType

Memory entries are classified by type:

- `'task'` — Task-related information
- `'context'` — Contextual knowledge
- `'event'` — Discrete events
- `'task-start'` — Task initiation markers
- `'task-complete'` — Task completion markers
- `'workflow-state'` — Workflow state snapshots
- Custom string types are also supported

### Methods

**Embedding Introspection**:

- `hasEmbedding(): boolean` — Returns whether this memory has an associated vector embedding.
- `getEmbeddingDimension(): number | undefined` — Returns the dimensionality of the embedding, or `undefined` if no embedding is set.

**Mutation**:

- `updateContent(content: string): void` — Replaces the stored content.
- `setEmbedding(embedding: number[]): void` — Sets or replaces the vector embedding.
- `updateMetadata(metadata: Record<string, unknown>): void` — Merges new metadata into the existing metadata.

**Query**:

- `matches(query: MemoryQuery): boolean` — Pattern matching by `agentId`, `type`, and `id`. Returns `true` if the memory satisfies the query constraints.
- `getAge(): number` — Returns milliseconds elapsed since creation.

### Factory Methods

Static factory methods provide ergonomic construction for common memory types:

```typescript
MemoryEntity.createTaskMemory(agentId, content, metadata?)
MemoryEntity.createContextMemory(agentId, content, metadata?)
MemoryEntity.createEventMemory(agentId, content, metadata?)
```

Each factory sets the appropriate `type`, generates a unique `id`, and timestamps the memory automatically.

---

## MemoryBackend Interface

All storage backends implement a common interface:

```typescript
interface MemoryBackend {
  initialize(): Promise<void>;
  close(): Promise<void>;

  store(memory: Memory): Promise<void>;
  retrieve(id: string): Promise<Memory | undefined>;
  update(memory: Memory): Promise<void>;
  delete(id: string): Promise<void>;

  query(query: MemoryQuery): Promise<Memory[]>;
  vectorSearch(embedding: number[], k: number): Promise<Memory[]>;
  clearAgent(agentId: string): Promise<void>;
}
```

### MemoryQuery

The `MemoryQuery` type supports filtering by:

| Field | Type | Description |
|-------|------|-------------|
| `agentId` | `string` | Filter by owning agent |
| `type` | `MemoryType` | Filter by memory type |
| `timeRange` | `{ start: Date; end: Date }` | Filter by creation time window |
| `metadata` | `Record<string, unknown>` | Filter by metadata key-value pairs |
| `limit` | `number` | Maximum number of results |
| `offset` | `number` | Pagination offset |

---

## Backend Implementations

### SQLiteBackend

Relational storage backed by SQLite. Provides structured queries with full SQL expressiveness.

- Efficient for metadata filtering, time-range queries, and agent-scoped retrieval
- Schema includes indexed columns for `agentId`, `type`, and `timestamp`
- Does not support native vector search (falls back to brute-force if called)

### AgentDBBackend

Vector-native storage with HNSW (Hierarchical Navigable Small World) indexing.

- **150x–12,500x faster** nearest-neighbor search compared to brute-force
- Optimized for high-dimensional embedding storage and retrieval
- Supports configurable HNSW parameters (M, efConstruction, efSearch)
- Best suited for semantic similarity queries

### HybridBackend (Default)

The recommended backend, selected by default per **ADR-009**. Combines SQLiteBackend and AgentDBBackend to provide the best of both worlds.

**Storage behavior**:
- All memories are stored in SQLite for structured queries
- Memories with embeddings are additionally stored in AgentDB for vector search

**Query routing**:
- Structured queries (`query()`) are routed to SQLite
- Vector search (`vectorSearch()`) is routed to AgentDB

**Lifecycle**:
- Both backends are initialized in parallel during `initialize()`
- Both backends are closed in parallel during `close()`

```
┌─────────────────────────────────────┐
│           HybridBackend             │
│                                     │
│  store(memory) ──┬──► SQLiteBackend │
│                  │    (all memories) │
│                  │                  │
│                  └──► AgentDBBackend│
│                       (if embedding)│
│                                     │
│  query()    ────────► SQLiteBackend │
│  vectorSearch() ────► AgentDBBackend│
└─────────────────────────────────────┘
```

---

## @claude-flow/memory Package

The full memory package extends the core backends with additional components:

| File | Description |
|------|-------------|
| `hnsw-index.ts` | HNSW vector indexing implementation for sub-millisecond nearest-neighbor search |
| `cache-manager.ts` | Caching layer for frequently accessed memories |
| `query-builder.ts` | Fluent query construction API |
| `migration.ts` | Schema migration management for database upgrades |
| `database-provider.ts` | Database connection abstraction and pooling |
| `sqljs-backend.ts` | Cross-platform WASM SQLite backend via sql.js (no native compilation required) |

### sql.js Backend

The `sqljs-backend.ts` provides a fully cross-platform SQLite implementation using sql.js (compiled to WebAssembly). This eliminates the need for native SQLite bindings, making the memory system portable across:

- Node.js (all platforms)
- Browser environments
- Edge runtimes
- Environments without native compilation toolchains

---

## @claude-flow/embeddings Package

The embeddings package provides vector embedding generation, processing, and storage.

| File | Description |
|------|-------------|
| `embedding-service.ts` | Core embedding generation service |
| `chunking.ts` | Document chunking with configurable overlap and chunk size |
| `normalization.ts` | Vector normalization (L2, L1, min-max, z-score) |
| `hyperbolic.ts` | Poincaré ball model for hierarchical data embeddings |
| `persistent-cache.ts` | Persistent embedding cache to avoid re-computation |
| `neural-integration.ts` | RuVector neural substrate integration |

### Normalization Strategies

| Strategy | Use Case |
|----------|----------|
| **L2** (Euclidean) | Default; unit-length vectors for cosine similarity |
| **L1** (Manhattan) | Sparse data, robust to outliers |
| **Min-Max** | Bounded [0, 1] range normalization |
| **Z-Score** | Statistical standardization (mean=0, stddev=1) |

### Hyperbolic Embeddings

The Poincaré ball model (`hyperbolic.ts`) encodes hierarchical relationships in hyperbolic space. This is particularly effective for:

- Code module hierarchies (packages → modules → classes → methods)
- Task dependency trees
- Agent organizational structures

Hyperbolic space naturally represents tree-like structures with exponentially more room at the edges, making it ideal for embedding hierarchical code relationships.

### Performance

With agentic-flow ONNX integration, embedding generation achieves **75x speedup** compared to baseline implementations.

---

## Design Decisions

| Decision | Rationale |
|----------|-----------|
| Hybrid backend as default | Ensures both structured queries and semantic vector search are available without configuration |
| HNSW indexing | Sub-millisecond nearest-neighbor search at scale; well-studied algorithm with predictable performance |
| sql.js for cross-platform support | WASM compilation eliminates native dependency issues across environments |
| Multiple normalization strategies | Different embedding models and use cases benefit from different normalization approaches |
| Poincaré embeddings | Hierarchical code relationships are naturally tree-like; hyperbolic space represents these more faithfully than Euclidean space |
| Persistent embedding cache | Embedding generation is expensive; caching avoids redundant computation across sessions |
