# Auxiliary Assets and Harnesses
Last updated: 2026-03-21

## Purpose
This is the support-layer map for the non-core assets that make the Claude Flow workspace usable: runtime helpers, plugin/install surfaces, agent archetypes, CI workflows, security gates, and container regression harnesses. It is intentionally scoped to implementation and manifests, not the archived module docs.

## Status
| Area | Status | Notes |
|---|---|---|
| This document | supporting asset | Sourced from live manifests and scripts only. |
| `.claude/` runtime layer | active | Current Claude Code runtime surface, statusline, hook handlers, and MCP wiring. |
| `.claude-plugin/` package | legacy | Checked-in 2.5.0 plugin distribution surface and installer. |
| `plugin/` surface | experimental | Newer wrapper/hook surface; only the hook manifest is present in-tree. |
| `agents/` archetypes | supporting asset | Small YAML agent set used by the live runtime. |
| `.github/` workflows | active | Verification, integration, rollback, and badge automation. |
| `.githooks/` | active | Pre-commit redaction gate. |
| `scripts/` | supporting asset | Installer, appliance verifier, and cleanup helper scripts. |
| `tests/docker-regression/` | supporting asset | Docker harness with broad smoke coverage and placeholder-style assertions. |
| `docs/ruvector-postgres/` | historical/unverified | No checked-in path was found in this workspace. |

## Covered Areas
| Area | What It Contains | Representative Files |
|---|---|---|
| Live runtime helpers | Statusline rendering, hook dispatch, memory/learning plumbing, and MCP bootstrap helpers. | `[.claude/statusline.sh](../../.claude/statusline.sh)`, `[.claude/statusline.mjs](../../.claude/statusline.mjs)`, `[.claude/helpers/hook-handler.cjs](../../.claude/helpers/hook-handler.cjs)`, `[.claude/helpers/setup-mcp.sh](../../.claude/helpers/setup-mcp.sh)` |
| Live runtime manifests | Runtime MCP config and performance/dependency tuning. | `[.claude/mcp.json](../../.claude/mcp.json)`, `[.claude/config/v3-performance-targets.json](../../.claude/config/v3-performance-targets.json)`, `[.claude/config/v3-dependency-optimization.json](../../.claude/config/v3-dependency-optimization.json)` |
| Legacy plugin package | Older distribution package metadata, docs, and hook manifest. | `[.claude-plugin/plugin.json](../../.claude-plugin/plugin.json)`, `[.claude-plugin/hooks/hooks.json](../../.claude-plugin/hooks/hooks.json)`, `[.claude-plugin/scripts/install.sh](../../.claude-plugin/scripts/install.sh)` |
| Newer wrapper surface | Minimal plugin hook manifest with richer hook event mappings. | `[plugin/hooks/hooks.json](../../plugin/hooks/hooks.json)` |
| Agent archetypes | YAML archetypes for the live agent catalog. | `[agents/architect.yaml](../../agents/architect.yaml)`, `[agents/coder.yaml](../../agents/coder.yaml)`, `[agents/reviewer.yaml](../../agents/reviewer.yaml)`, `[agents/security-architect.yaml](../../agents/security-architect.yaml)`, `[agents/tester.yaml](../../agents/tester.yaml)` |
| CI and verification | Workflow coverage for verification, integration, rollback, status badges, and V3-only checks. | `[.github/workflows/verification-pipeline.yml](../../.github/workflows/verification-pipeline.yml)`, `[.github/workflows/integration-tests.yml](../../.github/workflows/integration-tests.yml)`, `[.github/workflows/rollback-manager.yml](../../.github/workflows/rollback-manager.yml)`, `[.github/workflows/status-badges.yml](../../.github/workflows/status-badges.yml)`, `[.github/workflows/ci.yml](../../.github/workflows/ci.yml)`, `[.github/workflows/v3-ci.yml](../../.github/workflows/v3-ci.yml)` |
| Local scripts | Install, verify, and cleanup helpers for the current workspace. | `[scripts/install.sh](../../scripts/install.sh)`, `[scripts/verify-appliance.sh](../../scripts/verify-appliance.sh)`, `[scripts/cleanup-v3.sh](../../scripts/cleanup-v3.sh)` |
| Docker regression harness | End-to-end container test runner plus category scripts and fixtures. | `[tests/docker-regression/README.md](../../tests/docker-regression/README.md)`, `[tests/docker-regression/docker-compose.yml](../../tests/docker-regression/docker-compose.yml)`, `[tests/docker-regression/scripts/run-all-tests.sh](../../tests/docker-regression/scripts/run-all-tests.sh)` |
| Security gate | Git hook that blocks commits when the compiled redaction hook is available. | `[.githooks/pre-commit](../../.githooks/pre-commit)` |

## Key Entry Points
| Entry Point | Role | Status |
|---|---|---|
| `[.claude/statusline.sh](../../.claude/statusline.sh)` | Primary shell statusline for V3 metrics, swarm activity, memory, security, and context window state. | active |
| `[.claude/statusline.mjs](../../.claude/statusline.mjs)` | Alternate statusline implementation that shells out to `agentic-flow@alpha`. | experimental |
| `[.claude/helpers/hook-handler.cjs](../../.claude/helpers/hook-handler.cjs)` | Cross-platform hook dispatcher for route, pre/post task, session, and edit events. | active |
| `[.claude/helpers/setup-mcp.sh](../../.claude/helpers/setup-mcp.sh)` | Minimal MCP bootstrap that adds only the `claude-flow` server. | active |
| `[.claude/mcp.json](../../.claude/mcp.json)` | Runtime MCP configuration currently pinned to a `flow-nexus` command path. | active |
| `[.claude-plugin/plugin.json](../../.claude-plugin/plugin.json)` | Legacy plugin manifest with 2.5.0 metadata and three MCP server declarations. | legacy |
| `[.claude-plugin/scripts/install.sh](../../.claude-plugin/scripts/install.sh)` | Interactive installer that copies commands/agents and can write Claude MCP config. | legacy |
| `[plugin/hooks/hooks.json](../../plugin/hooks/hooks.json)` | Newer hook manifest with richer event mapping and command-based hook handlers. | experimental |
| `[scripts/install.sh](../../scripts/install.sh)` | Current installer branded as Ruflo, with MCP setup and quickstart output. | active |
| `[scripts/verify-appliance.sh](../../scripts/verify-appliance.sh)` | Comprehensive capability verifier for local appliance checks. | supporting asset |
| `[tests/docker-regression/scripts/run-all-tests.sh](../../tests/docker-regression/scripts/run-all-tests.sh)` | Main regression orchestrator for the Docker harness. | supporting asset |

## How It Works
| Layer | Behavior |
|---|---|
| Runtime layer | `.claude/statusline.sh` reads live metrics from `.claude-flow/*` state files, falls back to process and filesystem heuristics, and renders a compact operational summary. |
| Hook layer | `.claude/helpers/hook-handler.cjs` dispatches hook events such as `route`, `pre-bash`, `post-edit`, `pre-task`, and `session-end`; the hook manifests in `.claude-plugin/` and `plugin/` invoke `npx claude-flow@alpha` commands around those events. |
| MCP layer | The live runtime config in `.claude/mcp.json` and `.claude/helpers/setup-mcp.sh` do not describe the same server set, so MCP setup is split across the runtime layer, the legacy plugin manifest, and the newer hook surface. |
| Installation layer | `scripts/install.sh` installs the current Ruflo package and can add MCP servers, while `.claude-plugin/scripts/install.sh` targets the older plugin distribution flow. |
| Verification layer | `.github/workflows/*.yml` provides CI, integration, rollback, and badge automation; the Docker harness under `tests/docker-regression/` provides the broader regression pass. |

## Why It Is Designed This Way
| Design Choice | Reason |
|---|---|
| Separate runtime and plugin surfaces | The repo is carrying both a live Claude Code runtime and older/newer plugin packaging paths, so the support assets are split to keep local execution, installer behavior, and CI checks decoupled. |
| File-backed statusline and hooks | Shell/Node helpers can be invoked directly by Claude Code without waiting on a separate service, which keeps the runtime lightweight. |
| Command-driven hook manifests | The hook JSON files are declarative entry points; they let the runtime swap behavior without hardcoding every event in one place. |
| Container regression harness | The Docker suite provides a reproducible environment for broad checks, which is useful when live hooks and MCP setup vary by machine. |
| Redaction-first pre-commit hook | The repo protects secrets before code is committed, which is a low-friction guardrail for a project that stores many generated scripts. |

## Dependencies
| Dependency | Where It Shows Up | Notes |
|---|---|---|
| Node.js 20+ | `[.claude-plugin/plugin.json](../../.claude-plugin/plugin.json)`, `[scripts/install.sh](../../scripts/install.sh)`, `[.github/workflows/verification-pipeline.yml](../../.github/workflows/verification-pipeline.yml)` | The main manifests and CI target Node 20. |
| Claude Code CLI | `[.claude-plugin/scripts/install.sh](../../.claude-plugin/scripts/install.sh)`, `[scripts/install.sh](../../scripts/install.sh)` | Used for plugin installation and MCP registration. |
| `npx claude-flow@alpha` | `[.claude-plugin/plugin.json](../../.claude-plugin/plugin.json)`, `[.claude-plugin/hooks/hooks.json](../../.claude-plugin/hooks/hooks.json)`, `[plugin/hooks/hooks.json](../../plugin/hooks/hooks.json)` | The plugin and hook surfaces still target the alpha package name. |
| `npx flow-nexus@latest` | `[.claude-plugin/plugin.json](../../.claude-plugin/plugin.json)` | Optional cloud MCP server in the legacy plugin manifest. |
| Docker / Compose | `[tests/docker-regression/README.md](../../tests/docker-regression/README.md)`, `[tests/docker-regression/docker-compose.yml](../../tests/docker-regression/docker-compose.yml)` | Required for the regression harness. |
| `jq`, `sqlite3`, `bc`, `timeout`, `nc` | `[.claude/statusline.sh](../../.claude/statusline.sh)`, `[scripts/verify-appliance.sh](../../scripts/verify-appliance.sh)`, `[tests/docker-regression/docker-compose.yml](../../tests/docker-regression/docker-compose.yml)` | Several helpers and harness scripts assume these Unix tools are present. |
| Git | `[.githooks/pre-commit](../../.githooks/pre-commit)`, `[scripts/cleanup-v3.sh](../../scripts/cleanup-v3.sh)` | Needed for commit gating and cleanup/tracking operations. |

## Operational/Test Notes
| Note | Evidence |
|---|---|
| The live runtime hook handler is permissive by default and mostly records or routes events; it does not implement every security expectation that the test harness asserts. | `[.claude/helpers/hook-handler.cjs](../../.claude/helpers/hook-handler.cjs)`, `[tests/docker-regression/scripts/test-security.sh](../../tests/docker-regression/scripts/test-security.sh)` |
| The pre-commit hook only runs a compiled redaction validator when `dist-cjs/src/hooks/redaction-hook.js` exists. | `[.githooks/pre-commit](../../.githooks/pre-commit)` |
| The Docker regression scripts lean heavily on smoke checks and `|| echo ...` fallbacks, so they verify command shapes as much as end-to-end behavior. | `[tests/docker-regression/scripts/test-hooks.sh](../../tests/docker-regression/scripts/test-hooks.sh)`, `[tests/docker-regression/scripts/test-security.sh](../../tests/docker-regression/scripts/test-security.sh)`, `[tests/docker-regression/scripts/test-plugins.sh](../../tests/docker-regression/scripts/test-plugins.sh)`, `[tests/docker-regression/scripts/run-integration-tests.sh](../../tests/docker-regression/scripts/run-integration-tests.sh)` |
| The appliance verifier is broader than the Docker harness and is intended as a local capability sweep, not a strict unit-test substitute. | `[scripts/verify-appliance.sh](../../scripts/verify-appliance.sh)` |
| The current runtime MCP bootstrap only registers `claude-flow`, while the checked-in runtime MCP JSON names `flow-nexus` with a fixed path. | `[.claude/helpers/setup-mcp.sh](../../.claude/helpers/setup-mcp.sh)`, `[.claude/mcp.json](../../.claude/mcp.json)` |

## Known Drift
| Drift | Evidence | Why It Matters |
|---|---|---|
| Version skew across surfaces | `.claude-plugin` is pinned to 2.5.0, the Docker regression suite writes `version: 3.0.0`, and the live hook manifests still call `claude-flow@alpha`. | It is unclear which packaging line is authoritative for release and verification work. |
| Missing install inputs for the legacy plugin flow | `.claude-plugin/scripts/install.sh` expects `commands/` and `agents/` under the plugin directory, but the checked-in `plugin/` surface only contains `hooks/hooks.json`, and the workspace has no checked-in `docs/ruvector-postgres/` tree. | A fresh install from the legacy package path can fail or silently produce an incomplete surface. |
| Conflicting MCP wiring | `.claude/mcp.json` only defines `flow-nexus`, `.claude/helpers/setup-mcp.sh` only adds `claude-flow`, and both hook manifests target `claude-flow@alpha`. | There is no single canonical MCP configuration for the workspace. |
| Missing truth-scoring workflow target | `.github/workflows/status-badges.yml` and `.github/workflows/rollback-manager.yml` both reference `🎯 Truth Scoring Pipeline`, but no matching workflow file is present in the checked-in workflow set. | Badge and rollback automation can report on a workflow that is not actually defined here. |
| Legacy branding remains in active files | `.claude/statusline-command.sh` rewrites `claude-code-flow`, and `status-badges.yml` still points at `ruvnet/claude-code-flow`. | The repo still carries the old brand in user-facing automation. |
| Regression harness expectations exceed live behavior | `test-security.sh` asserts broad path-traversal, injection, and CVE blocking, while the live hook handler only blocks a short hard-coded dangerous-command list and the pre-commit hook only performs redaction if a compiled validator exists. | The harness can suggest stronger security coverage than the runtime currently delivers. |

## Related Docs
| Document | Relevance |
|---|---|
| `[docs/prompt.txt](../prompt.txt)` | Workspace-level doc-generation prompt that references this architecture note. |
| `[.claude/helpers/README.md](../../.claude/helpers/README.md)` | Overview of the live helper scripts and their roles. |
| `[.claude-plugin/docs/PLUGIN_SUMMARY.md](../../.claude-plugin/docs/PLUGIN_SUMMARY.md)` | Legacy plugin summary and manifest overview. |
| `[.claude-plugin/docs/INSTALLATION.md](../../.claude-plugin/docs/INSTALLATION.md)` | Legacy install guidance for the 2.5.0 plugin surface. |
| `[tests/docker-regression/README.md](../../tests/docker-regression/README.md)` | Docker regression harness overview and usage. |
| `[.github/ISSUE_PATTERN_PERSISTENCE.md](../../.github/ISSUE_PATTERN_PERSISTENCE.md)` | Example of workflow-linked pattern persistence and learning references. |
