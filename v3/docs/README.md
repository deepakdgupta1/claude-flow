# V3 Docs Bridge
Last updated: 2026-03-21

## Purpose
This page restores the `v3/README.md` documentation links by bridging them to the fresh repo-level architecture corpus and the implementation notes that still live under `v3/`.

## Status
| Surface | Status | Notes |
|---|---|---|
| This bridge page | `supporting asset` | Compatibility layer for `v3/README.md`. |
| Fresh architecture corpus | `active` | The authoritative narrative now lives under `../../docs/architecture/`. |
| Existing `v3/docs/adr` and `v3/docs/ddd` trees | `supporting asset` | Historical/package-local implementation notes still worth linking. |
| Missing pre-existing `v3/docs/guides` and `v3/docs/implementation` paths | `legacy` | Recreated as bridge directories so the old links resolve again. |

## Covered Areas
| Topic | Where to go |
|---|---|
| Repo-level entrypoint | [`../../docs/README.md`](../../docs/README.md) |
| Master V3 architecture narrative | [`../../docs/architecture/README.md`](../../docs/architecture/README.md) |
| Runtime ownership | [`../../docs/architecture/v3-core-runtime.md`](../../docs/architecture/v3-core-runtime.md) |
| CLI, MCP, hooks, operations, and `ruflo` | [`../../docs/architecture/v3-surfaces-and-operations.md`](../../docs/architecture/v3-surfaces-and-operations.md) |
| Security and governance | [`../../docs/architecture/v3-security-governance.md`](../../docs/architecture/v3-security-governance.md) |
| Plugin ecosystem | [`../../docs/architecture/v3-plugin-ecosystem.md`](../../docs/architecture/v3-plugin-ecosystem.md) |
| ADR material already in `v3/` | [`../implementation/adrs/README.md`](../implementation/adrs/README.md), [`./adr/README.md`](./adr/README.md) |
| Existing DDD notes | [`./ddd/coherence-engine/README.md`](./ddd/coherence-engine/README.md), [`./ddd/quality-engineering/README.md`](./ddd/quality-engineering/README.md) |
| Guide bridge | [`./guides/README.md`](./guides/README.md) |
| Implementation bridge | [`./implementation/README.md`](./implementation/README.md) |

## Key Entry Points
| Path | Role |
|---|---|
| [`../../docs/README.md`](../../docs/README.md) | Main documentation entrypoint for the repository. |
| [`../../docs/architecture/README.md`](../../docs/architecture/README.md) | Master architecture narrative for the active runtime. |
| [`../README.md`](../README.md) | Historical V3 overview that points here. |
| [`../implementation/adrs/README.md`](../implementation/adrs/README.md) | Detailed ADR collection that still lives inside `v3/`. |

## How It Works
This page does not duplicate the new architecture corpus. Instead, it routes readers by intent:

1. Start with the fresh repo-level docs under `../../docs/`.
2. Use the bridge pages under `./guides/` and `./implementation/` when arriving from `v3/README.md`.
3. Drop into the package-local ADR and DDD material when you need historical implementation detail rather than the current architecture story.

## Why It Is Designed This Way
The older `v3/README.md` already points into `v3/docs/`. Recreating those paths as bridges fixes broken links without forcing the repo to maintain two competing full documentation trees inside `v3/`.

## Dependencies
| Dependency | Notes |
|---|---|
| Relative links into `../../docs/architecture/` | The fresh architecture corpus is the primary destination. |
| Existing `v3/docs/adr` and `v3/docs/ddd` trees | Still used for local implementation notes. |
| Existing `v3/implementation/adrs/` tree | Remains the source of ADR detail. |

## Operational/Test Notes
Keep this page light and link-oriented. If a concept is about current architecture truth, update `../../docs/architecture/` first and only adjust this page when the destination links change.

## Known Drift
| Drift | Why it matters |
|---|---|
| `v3/README.md` still describes a simpler 10-module story than the current scoped package tree. | Readers arriving here should be routed to the fresh architecture corpus for the current package map. |
| The historical `v3/docs/` tree was only partially present in this checkout. | This bridge restores the expected link targets without pretending the old tree is the full current source of truth. |

## Related Docs
| Document | Why it matters |
|---|---|
| [`../../docs/README.md`](../../docs/README.md) | Repo-level docs entrypoint. |
| [`../../docs/architecture/README.md`](../../docs/architecture/README.md) | Master architecture narrative. |
| [`./guides/README.md`](./guides/README.md) | Goal-oriented guide bridge. |
| [`./implementation/README.md`](./implementation/README.md) | Implementation-detail bridge. |
