# v2 Legacy Architecture

Last updated: 2026-03-21

Status: `legacy`

This document describes Claude-Flow v2 as implemented in the codebase, not as the older architecture narratives described it. v2 remains important for migration, support, and compatibility context, but new architecture work should target the v3 runtime.

## Purpose
Claude-Flow v2 is the legacy execution line for existing installs and project migrations. It still matters because it shows how the current package boots, how orchestration is split, and where compatibility boundaries live.

## Status
- `legacy`: v2 is the compatibility baseline and should be treated as historical runtime context.
- `active`: the package is still shipped as `claude-flow` 2.7.47 in [`../../v2/package.json`](../../v2/package.json).
- `experimental`: the MCP 2025-11 and progressive disclosure path exists, but it is layered on top of the older server model.
- `supporting asset`: migration, verification, and benchmark material help preserve the v2 support story.
- `historical/unverified`: older feature counts, architecture summaries, and marketing-style claims should not be treated as authoritative contracts.

## Covered Areas
- `../../v2/package.json` and `../../v2/README.md`
- `../../v2/src/core/*` orchestration glue
- `../../v2/src/cli/*` Node CLI, Deno/simple CLI shim, and command dispatch
- `../../v2/src/mcp/*` MCP transport, registry, progressive disclosure, and protocol negotiation
- `../../v2/src/swarm/*` swarm execution, prompt copying, and swarm-local memory
- `../../v2/src/hive-mind/*` queen-led coordination, consensus, and hive-local memory
- `../../v2/src/providers/*` provider adapters and provider manager
- `../../v2/src/reasoningbank/*` ReasoningBank-backed legacy memory adapter
- `../../v2/src/verification/*` truth scoring, security enforcement, rollback, and hooks
- `../../v2/src/enterprise/*` deployment, audit, cloud, analytics, and security managers
- `../../v2/src/migration/*` prompt/config/assets migration tooling

## Key Entry Points
- [`../../v2/package.json`](../../v2/package.json) wires `main: cli.mjs` and the `claude-flow` binary entry point.
- [`../../v2/src/cli/main.ts`](../../v2/src/cli/main.ts) is the Node CLI bootstrap.
- [`../../v2/src/cli/simple-cli.ts`](../../v2/src/cli/simple-cli.ts) is the Deno-compatible simple CLI shim.
- [`../../v2/bin/claude-flow.js`](../../v2/bin/claude-flow.js) is the packaged binary dispatcher.
- [`../../v2/src/index.js`](../../v2/src/index.js) is the Express API entry point.
- [`../../v2/src/mcp/index.ts`](../../v2/src/mcp/index.ts) is the MCP facade and integration factory.
- [`../../v2/src/mcp/server.ts`](../../v2/src/mcp/server.ts) is the standard 2024.11.5 MCP server.
- [`../../v2/src/mcp/server-mcp-2025.ts`](../../v2/src/mcp/server-mcp-2025.ts) is the 2025-11 enhanced server path.
- [`../../v2/src/migration/index.ts`](../../v2/src/migration/index.ts) is the separate migration CLI.

## How It Works
The v2 package boots through several entry surfaces rather than one uniform runtime. The Node CLI and binary dispatcher route into the command registry, the Deno shim provides a fallback CLI path, and the Express entry exposes a small API surface.

Core orchestration lives in `v2/src/core` and coordinates swarm, hive-mind, MCP, memory, and monitoring concerns. The design is deliberately split, so no single module owns the entire runtime lifecycle.

MCP has two layers. The standard server in [`../../v2/src/mcp/server.ts`](../../v2/src/mcp/server.ts) speaks the 2024.11.5 protocol over stdio or HTTP. The newer path in [`../../v2/src/mcp/server-mcp-2025.ts`](../../v2/src/mcp/server-mcp-2025.ts) adds version negotiation, async jobs, registry integration, and schema validation. Progressive tool discovery in [`../../v2/src/mcp/tool-registry-progressive.ts`](../../v2/src/mcp/tool-registry-progressive.ts) loads tools lazily through `tools/search`.

Swarm and hive-mind are parallel coordination systems, not one shared abstraction. Swarm is task and execution oriented, with its own memory and prompt-copying helpers. Hive-mind is queen-led and consensus oriented, with its own memory and database layer. They overlap conceptually, but they are implemented as separate subsystems.

Memory is also split by subsystem. CLI commands, swarm memory, hive-mind memory, and the ReasoningBank adapter each manage their own persistence path. That split is part of why v2 is compatible with older projects, but it also means there is no single unified memory service in the legacy line.

Provider support is pluggable through [`../../v2/src/providers/index.ts`](../../v2/src/providers/index.ts), which exports Anthropic, OpenAI, Google, Cohere, and Ollama adapters through a shared provider manager.

Verification combines security enforcement, truth scoring, rollback, checkpoints, and hook-based integration. The pieces are cohesive at the subsystem level, but the public entry surface is still split across multiple files rather than one stable facade.

Enterprise support sits beside the runtime, not inside it. Deployment, audit, cloud, security, and analytics managers add operational structure without changing the orchestration model.

Migration is intentionally narrow. The migration package is for prompts, configs, backups, and asset reshaping, not for rewriting runtime architecture.

## Why It Is Designed This Way
v2 is optimized for incremental adoption and backward compatibility. The split entry points let the package run in Node, Deno-compatible CLI mode, and HTTP API mode without forcing one runtime shape on every user.

Separate swarm and hive-mind implementations preserve two different coordination models. That makes the system easier to adapt for short-lived task execution versus longer-lived consensus workflows.

The distributed memory model keeps legacy behavior intact while ReasoningBank and optional AgentDB-style paths evolve independently. That is messy, but it avoids breaking older projects that already depend on subsystem-local state.

Progressive MCP disclosure exists to reduce startup cost and tool-loading overhead. In practice, it is a compatibility and performance layer over a legacy MCP server, not a full replacement for the older command model.

## Dependencies
- Runtime: Node.js 20+ and the package-level binary wiring in [`../../v2/package.json`](../../v2/package.json).
- CLI and API: Commander, Deno-compatible shims, and Express/CORS for the API entry.
- MCP: `@modelcontextprotocol/sdk`, optional Claude Code SDK integration, and internal transport/registry modules.
- Orchestration: `agentic-flow` and `ruv-swarm` for swarm and hive-related coordination paths.
- Memory: ReasoningBank/SQLite support plus optional AgentDB-related dependencies in the package manifest.
- Verification and enterprise: internal managers for truth scoring, rollback, audit, deployment, and analytics.
- Migration: the standalone migration CLI in [`../../v2/src/migration/index.ts`](../../v2/src/migration/index.ts) and its backup/validation helpers.

## Operational/Test Notes
- Use `npm run typecheck` and `npm test` for broad validation.
- Use the subsystem tests that match the change: MCP tests under `v2/tests/mcp`, verification tests under `v2/src/verification/tests`, migration tests under `v2/src/migration/tests`, and CLI/integration tests under `v2/tests/integration`.
- Because entry points are split, smoke-test the exact surface you changed instead of relying on a single package-level run.
- Prefer verifying the actual exported path in code before updating docs, because legacy help text and runtime behavior are not always aligned.

## Known Drift
- README-level claims such as tool counts, agent counts, and capability totals are not backed by one authoritative runtime manifest.
- CLI help in [`../../v2/src/cli/simple-mcp.ts`](../../v2/src/cli/simple-mcp.ts) advertises `system/info` and `system/health`, while the real MCP server in [`../../v2/src/mcp/server.ts`](../../v2/src/mcp/server.ts) also registers `system/status`, `tools/list`, and `tools/schema`, and the progressive registry adds `tools/search`.
- The memory story is internally inconsistent across docs and subsystems: ReasoningBank, swarm memory, hive-mind memory, and AgentDB references coexist without a single unified v2 contract.
- Migration docs describe `claude-flow migrate ...` style flows, but the implementation is a separate `claude-flow-migrate` CLI in [`../../v2/src/migration/index.ts`](../../v2/src/migration/index.ts) that focuses on project prompts, configs, backups, and validation.
- Verification exports are split across multiple files, so consumers should not assume one stable top-level facade for security, truth scoring, and rollback behavior.
- Historical architecture notes in v2 are useful for context, but they should be treated as `historical/unverified` whenever they conflict with the code.

## Related Docs
- [`./README.md`](./README.md) is the current architecture baseline for the newer runtime.
- [`../../v2/README.md`](../../v2/README.md) is the legacy package overview.
- [`../../v2/src/mcp/README.md`](../../v2/src/mcp/README.md) covers the MCP subsystem in more detail.
- [`../../v2/src/verification/README.md`](../../v2/src/verification/README.md) and [`../../v2/src/verification/architecture.md`](../../v2/src/verification/architecture.md) cover verification and truth enforcement.
- [`../../v2/src/migration/README.md`](../../v2/src/migration/README.md) explains the migration CLI scope.
- [`../../v2/docs/architecture/README.md`](../../v2/docs/architecture/README.md) is historical v2 architecture material and should be used for contrast only.
