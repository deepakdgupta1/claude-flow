# Module 14: V2 Legacy System

**Location**: `v2/`

**Purpose**: The previous major version of Claude Flow. Maintained for backward compatibility and migration support. V3 provides a structured migration path from V2 with automated tooling, compatibility aliases, and rollback support.

---

## Overview

V2 was the original monolithic Claude Flow implementation. V3 introduced a modular architecture, new features (HNSW vector search, neural learning, plugin system), and breaking changes. The V2 codebase is preserved in its own directory for reference and to support gradual migration.

```
v2/
├── src/              # Source code (monolithic)
├── bin/              # CLI binary entry point
├── tests/            # Test suite (Jest-based)
├── docs/             # V2-specific documentation
├── hooks/            # V2 hook system
├── models/           # Data models
├── examples/         # Usage examples
├── scripts/          # Utility scripts
├── docker/           # Docker configuration
├── benchmark/        # Performance benchmarks
└── patches/          # Compatibility patches
```

---

## V2 Directory Structure

### `src/`
The main source code directory. V2 used a monolithic architecture where all functionality (agents, tasks, memory, coordination) lived in a single interconnected codebase. This contrasts with V3's modular package structure under `v3/@claude-flow/`.

### `bin/`
CLI binary entry point. V2's CLI was simpler with fewer commands. The binary bootstraps the runtime and parses command-line arguments.

### `tests/`
Test suite using Jest as the test runner. Tests follow the `*.test.ts` naming convention and are co-located with or mirror the source directory structure.

### `docs/`
V2-specific documentation. Includes API references, configuration guides, and architecture decisions that were relevant to the V2 design.

### `hooks/`
V2's hook system. Simpler than V3's 17-hook system — V2 supported a smaller set of lifecycle hooks with different naming conventions (see compatibility section below).

### `models/`
Data models used by V2. These define the shape of agents, tasks, and other domain objects. V3's shared types (`v3/src/shared/types/`) supersede these.

### `examples/`
Usage examples demonstrating V2 features. Useful as reference for understanding V2 patterns when migrating.

### `scripts/`
Utility scripts for development, deployment, and maintenance tasks.

### `docker/`
Docker configuration for containerized deployment. Includes Dockerfile and docker-compose configurations with test confirmation.

### `benchmark/`
Performance benchmark suite for measuring V2's execution characteristics. Results can be compared against V3 benchmarks to validate migration improvements.

### `patches/`
Compatibility patches applied to V2 code for interoperability with V3 components during the migration period.

---

## V2 Configuration & Tooling

V2 used a different tooling stack than V3:

| Concern | V2 | V3 |
|---|---|---|
| **Test Runner** | Jest | Vitest |
| **Transpilation** | SWC | TypeScript compiler (tsc) |
| **Linting** | ESLint + Prettier | ESLint + Prettier (shared config) |
| **Versioning** | Semantic Release | Manual + npm dist-tags |
| **Containerization** | Docker with test confirmation | Docker (optional) |
| **Module Format** | CommonJS | ESM |
| **Configuration** | JavaScript config files | JSON + TypeScript config |

### Jest Configuration
V2 tests run with Jest and SWC for fast transpilation:

```json
{
  "transform": {
    "^.+\\.(t|j)sx?$": "@swc/jest"
  },
  "testMatch": ["**/tests/**/*.test.ts"],
  "moduleFileExtensions": ["ts", "tsx", "js", "jsx", "json"]
}
```

### Docker Support
V2 included Docker-first deployment:

```bash
# Build V2 container
docker build -t claude-flow:v2 -f v2/docker/Dockerfile .

# Run with test confirmation
docker run claude-flow:v2 npm test && docker run claude-flow:v2 npm start
```

---

## Migration Path (V2 → V3)

The V3 CLI includes a dedicated migration command that automates the transition from V2 to V3.

### Migration CLI

```bash
npx @claude-flow/cli@latest migrate
```

#### `migrate status`

Check the current state and identify what needs migrating:

```bash
npx @claude-flow/cli@latest migrate status
```

**Output**:
```
Migration Status Report
========================
V2 installation found: v2/
V2 config files: 3 found
V2 hooks: 5 found (2 need renaming)
V2 data: SQLite database with 1,247 entries
V2 scripts: 4 found (3 compatible)

Recommended actions:
  ✓ Rename pre-bash → pre-command (2 hooks)
  ✓ Migrate SQLite data to hybrid backend
  ✓ Update config format (JSON → V3 schema)
  ⚠ Review custom scripts for V3 compatibility
```

#### `migrate run --backup`

Execute the migration with automatic backup:

```bash
npx @claude-flow/cli@latest migrate run --backup
```

This command:
1. Creates a timestamped backup of V2 data and configuration
2. Renames hooks to V3 conventions
3. Converts configuration files to V3 format
4. Migrates memory data to the V3 hybrid backend
5. Updates script references
6. Generates a migration report

**Flags**:
- `--backup`: Create backup before migrating (strongly recommended)
- `--dry-run`: Show what would be changed without making changes
- `--force`: Skip confirmation prompts

#### `migrate rollback`

Revert to V2 state if migration causes issues:

```bash
npx @claude-flow/cli@latest migrate rollback
```

Restores from the backup created during `migrate run --backup`. Only works if a backup exists.

#### `migrate validate`

Verify that migration completed successfully:

```bash
npx @claude-flow/cli@latest migrate validate
```

**Output**:
```
Migration Validation
=====================
Config format:     ✓ Valid V3 schema
Hook names:        ✓ All V3 conventions
Memory backend:    ✓ Hybrid backend operational
Data integrity:    ✓ 1,247/1,247 entries migrated
Script references: ✓ All updated
CLI commands:      ✓ All functional

Migration status: COMPLETE ✓
```

---

## V2 Compatibility in V3

V3 maintains backward compatibility with V2 patterns through aliases and compatibility layers.

### Hook Aliases

V2 hook names are automatically mapped to their V3 equivalents:

| V2 Hook | V3 Hook | Notes |
|---|---|---|
| `pre-bash` | `pre-command` | Triggered before command execution |
| `post-bash` | `post-command` | Triggered after command execution |
| `route-task` | `route` | Task routing/intelligence hook |
| `session-start` | `session-start` | Same name, but V3 supports additional session metadata |

When V2 hook names are used in V3, a deprecation warning is logged:

```
[WARN] Hook 'pre-bash' is deprecated. Use 'pre-command' instead.
       V2 compatibility alias will be removed in V4.
```

### Session Format Compatibility

The `session-start` hook in V3 accepts both V2 and V3 session formats:

```typescript
// V2 format (still accepted)
{
  sessionId: string;
  startTime: number; // Unix timestamp
}

// V3 format (preferred)
{
  sessionId: string;
  startedAt: Date;
  metadata: Record<string, unknown>;
  agentConfig: AgentConfig;
}
```

### SwarmHub Compatibility Layer

V2's `SwarmHub` class is maintained as a compatibility wrapper around V3's `UnifiedSwarmCoordinator`:

```typescript
// V2 code (still works)
import { SwarmHub } from 'claude-flow';
const hub = new SwarmHub(config);
await hub.start();

// Internally maps to V3:
// const coordinator = new UnifiedSwarmCoordinator(config);
// await coordinator.initialize();
```

**Deprecation notice**: `SwarmHub` is deprecated and will be removed in V4. New code should use `UnifiedSwarmCoordinator` directly.

---

## Key Differences: V2 vs V3

| Feature | V2 | V3 |
|---|---|---|
| **Architecture** | Monolithic | Modular (scoped packages) |
| **Memory Search** | Linear scan | HNSW vector index (150x–12,500x faster) |
| **Agent Types** | ~15 built-in | 60+ specialized types |
| **Hook System** | 6 hooks | 17 hooks + 12 background workers |
| **Plugin System** | None | Full plugin lifecycle + IPFS registry |
| **Neural Learning** | None | SONA + MoE + EWC++ |
| **Consensus** | Basic leader election | Byzantine, Raft, Gossip, CRDT, Quorum |
| **CLI Commands** | 8 commands | 26 commands, 140+ subcommands |
| **MCP Support** | Basic | Full MCP server with tool providers |
| **Type Safety** | Partial | Comprehensive shared type system |
| **Error Handling** | Generic errors | Typed error hierarchy with codes |

---

## Design Decisions

### V2 Preserved, Not Deleted
The V2 directory is kept in the repository rather than being removed. This serves multiple purposes:
- Reference implementation for understanding original design decisions
- Source of truth for migration tooling (needs to read V2 formats)
- Fallback if V3 migration needs to be reverted
- Historical record of the project's evolution

### Automated Migration Tool
The `migrate` CLI command automates the tedious parts of migration (renaming hooks, converting configs, migrating data) to reduce human error. The `--backup` flag and `rollback` command reduce migration risk.

### Compatibility Aliases
Hook aliases (`pre-bash` → `pre-command`) ensure existing V2 scripts continue working in V3 without modification. This follows the principle of not breaking working systems during upgrades. Deprecation warnings guide users toward V3 patterns without forcing immediate changes.

### Rollback Support
The migration system supports full rollback because data migration is inherently risky. If the V3 backend has issues with migrated data, users can revert to V2 quickly rather than debugging a partially-migrated state.

### Deprecation Timeline
V2 compatibility features follow a deprecation timeline:
- **V3.0**: Full V2 compatibility with deprecation warnings
- **V3.x**: Compatibility maintained, warnings become more prominent
- **V4.0**: V2 compatibility removed, V2 directory archived
