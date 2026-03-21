# CLI Module

## Overview

The CLI is the **primary user-facing interface** for Claude Flow, providing 26 commands with 140+ subcommands for managing agents, swarms, memory, workflows, security, plugins, and the full system lifecycle. It also doubles as the MCP server entry point — when stdin is piped (by Claude Code), it automatically starts in MCP mode.

**Location**: `v3/@claude-flow/cli/`

**Binary**: `claude-flow` (defined in root `package.json` → `v3/@claude-flow/cli/bin/cli.js`)

**Usage**: `npx claude-flow@alpha <command> [subcommand] [options]` or `npx @claude-flow/cli@latest <command> [options]`

---

## Architecture

```
bin/cli.js (entry point)
    ↓
src/parser.ts (command parsing)
    ↓
src/commands/<command>.ts (command handler)
    ↓
src/services/ (business logic) → src/memory/ → src/plugins/ → src/infrastructure/
    ↓
src/output.ts (formatted terminal output)
```

### Key Source Files

| File | Purpose |
|------|---------|
| `src/index.ts` | Main CLI module export |
| `src/parser.ts` | Command-line argument parsing |
| `src/output.ts` | Formatted terminal output (colors, tables, progress) |
| `src/mcp-server.ts` | MCP server mode entry point |
| `src/mcp-client.ts` | MCP client for connecting to external MCP servers |
| `src/config-adapter.ts` | Configuration loading, validation, and adaptation |
| `src/types.ts` | CLI-specific type definitions |
| `src/prompt.ts` | Interactive prompt utilities |
| `src/suggest.ts` | Command suggestion/autocomplete |

---

## Command Reference (26 Commands)

### Core Commands

| Command | Sub-commands | Description |
|---------|-------------|-------------|
| **`init`** | `--wizard`, `--preset`, `--skills`, `--hooks` | Project initialization with interactive wizard, preset templates, skill and hook installation |
| **`agent`** | `spawn`, `list`, `status`, `stop`, `metrics`, `pool`, `health`, `logs` | Full agent lifecycle management |
| **`swarm`** | `init`, `status`, `scale`, `stop`, `reconfigure`, `list` | Multi-agent swarm orchestration. Init accepts `--topology`, `--max-agents`, `--strategy` |
| **`memory`** | `init`, `store`, `retrieve`, `search`, `list`, `delete`, `clear`, `stats`, `export`, `import`, `migrate` | Full memory CRUD + vector search + import/export |
| **`mcp`** | `start`, `stop`, `status`, `list-tools`, `call`, `add-server`, `remove-server`, `test`, `logs` | MCP server and tool management |
| **`task`** | `create`, `assign`, `status`, `list`, `cancel`, `complete` | Task creation and lifecycle |
| **`session`** | `start`, `end`, `restore`, `list`, `status`, `export`, `delete` | Session state persistence across conversations |
| **`config`** | `get`, `set`, `list`, `validate`, `reset`, `export`, `import` | Configuration management |
| **`status`** | `--watch`, `--json`, `--detailed` | System status monitoring with live watch mode |
| **`start`** | `--quick`, `--daemon`, `--mcp` | Quick service startup |
| **`workflow`** | `run`, `list`, `status`, `pause`, `resume`, `cancel` | Workflow execution and management |
| **`hooks`** | (27 hooks + worker management) | See Hooks System module (07) for full details |
| **`hive-mind`** | `init`, `status`, `propose`, `vote`, `results`, `health` | Queen-led Byzantine fault-tolerant consensus |

### Advanced Commands

| Command | Sub-commands | Description |
|---------|-------------|-------------|
| **`daemon`** | `start`, `stop`, `status`, `trigger`, `enable` | Background worker daemon management |
| **`neural`** | `train`, `status`, `patterns`, `predict`, `optimize` | Neural pattern training and prediction |
| **`security`** | `scan`, `audit`, `cve`, `threats`, `validate`, `report` | Security scanning and vulnerability detection |
| **`performance`** | `benchmark`, `profile`, `metrics`, `optimize`, `report` | Performance profiling and optimization |
| **`providers`** | `list`, `add`, `remove`, `test`, `configure` | AI provider management (Anthropic, OpenAI, Google, Cohere, Ollama) |
| **`plugins`** | `list`, `install`, `uninstall`, `enable`, `disable` | Plugin management with IPFS marketplace integration |
| **`deployment`** | `deploy`, `rollback`, `status`, `environments`, `release` | Deployment lifecycle management |
| **`embeddings`** | `embed`, `batch`, `search`, `init` | Vector embedding operations (75x faster with agentic-flow ONNX) |
| **`claims`** | `check`, `grant`, `revoke`, `list` | Claims-based authorization management |
| **`migrate`** | `status`, `run`, `rollback`, `validate`, `plan` | V2→V3 migration with backup and rollback |
| **`process`** | `list`, `start`, `stop`, `logs` | Background process management |
| **`doctor`** | `--fix` | System diagnostics and health checks |
| **`completions`** | `bash`, `zsh`, `fish`, `powershell` | Shell completion script generation |

### Utility Commands

| Command | Description |
|---------|-------------|
| **`analyze`** | Codebase analysis |
| **`issues`** | Issue tracking integration |
| **`progress`** | Implementation progress tracking |
| **`route`** | Task routing to optimal agents |
| **`transfer-store`** | Pattern transfer between projects |
| **`update`** | Self-update mechanism |

---

## Doctor Health Checks

`npx @claude-flow/cli doctor` validates:

| Check | What It Verifies |
|-------|------------------|
| Node.js version | ≥20.0.0 |
| npm version | ≥9.0.0 |
| Git | Installed and accessible |
| Config file | Valid JSON, required fields present |
| Daemon | Running and responsive |
| Memory database | SQLite/AgentDB accessible |
| API keys | Anthropic/OpenAI/Google keys configured |
| MCP servers | Registered and responsive |
| Disk space | Sufficient for operation |
| TypeScript | Installed for build operations |

`--fix` flag attempts automatic remediation of detected issues.

---

## Supporting Infrastructure

| Directory | Purpose |
|-----------|---------|
| `src/services/` | Business logic services (separated from command handlers) |
| `src/memory/` | CLI-side memory management and persistence |
| `src/plugins/` | Plugin discovery, marketplace (IPFS/Pinata), store integration |
| `src/infrastructure/` | Runtime infrastructure and environment detection |
| `src/runtime/` | Runtime management and process control |
| `src/ruvector/` | RuVector intelligence integration for CLI |
| `src/init/` | Project initialization templates, wizards, preset configurations |
| `src/production/` | Production deployment utilities and health checking |
| `src/transfer/` | Pattern transfer between projects (IPFS-based) |
| `src/update/` | Self-update mechanism |
| `src/mcp-tools/` | MCP tool definitions exposed by the CLI |

---

## MCP Auto-Detection

The CLI implements a dual-mode architecture:

1. **Interactive CLI mode**: When run from a terminal, provides the full command-line interface
2. **MCP server mode**: When stdin is piped (detected automatically), starts as an MCP server exposing all capabilities as tools

This means a single `npx` invocation serves both purposes:
```bash
# Interactive CLI
npx @claude-flow/cli@latest agent list

# MCP server (when piped by Claude Code)
claude mcp add claude-flow -- npx -y @claude-flow/cli@alpha
```

---

## Configuration

The CLI loads configuration from multiple sources (in priority order):
1. Command-line flags (highest priority)
2. Environment variables (`CLAUDE_FLOW_*`)
3. Project config file (`claude-flow.config.json`)
4. Default values

Key environment variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `CLAUDE_FLOW_CONFIG` | `./claude-flow.config.json` | Config file path |
| `CLAUDE_FLOW_LOG_LEVEL` | `info` | Logging level |
| `CLAUDE_FLOW_MCP_PORT` | `3000` | MCP HTTP port |
| `CLAUDE_FLOW_MCP_HOST` | `localhost` | MCP host |
| `CLAUDE_FLOW_MCP_TRANSPORT` | `stdio` | MCP transport type |
| `CLAUDE_FLOW_MEMORY_BACKEND` | `hybrid` | Memory backend |
| `CLAUDE_FLOW_MEMORY_PATH` | `./data/memory` | Memory storage path |
| `ANTHROPIC_API_KEY` | — | Anthropic API key |
| `OPENAI_API_KEY` | — | OpenAI API key |
| `GOOGLE_API_KEY` | — | Google API key |

---

## Design Decisions

| Decision | Rationale |
|----------|-----------|
| Single binary for CLI + MCP | Eliminates separate processes; simpler deployment |
| MCP auto-detection via stdin | Zero-config integration with Claude Code |
| 26 command groups | Organized by domain for discoverability |
| Doctor command | Pre-flight checks prevent cryptic runtime errors |
| Migration tool built-in | V2→V3 migration without external tooling |
| Shell completions | Improves CLI UX for power users |
| Plugin marketplace in CLI | First-class plugin discovery and installation |
| Config priority cascade | Flexible configuration without file-first dependency |
