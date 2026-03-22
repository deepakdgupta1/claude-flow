# V3 Guides Bridge
Last updated: 2026-03-21

## Purpose
This page provides a stable `v3/docs/guides/` landing page for readers coming from `v3/README.md` and routes them to the new architecture guides that are backed by the current checkout.

## Status
| Surface | Status | Notes |
|---|---|---|
| This bridge page | `supporting asset` | Compatibility guide index for the old `v3/README.md` links. |
| Fresh architecture guides | `active` | Current guidance now lives under `../../../docs/architecture/`. |

## Covered Areas
| Reader goal | Recommended page |
|---|---|
| Understand what the product is and who it is for | [`../../../docs/architecture/README.md`](../../../docs/architecture/README.md) |
| See the repository layout and package/plugin coverage | [`../../../docs/architecture/repository-map.md`](../../../docs/architecture/repository-map.md) |
| Follow the request path through CLI, MCP, hooks, providers, and operations | [`../../../docs/architecture/v3-surfaces-and-operations.md`](../../../docs/architecture/v3-surfaces-and-operations.md) |
| Understand swarm, memory, integration, neural runtime ownership | [`../../../docs/architecture/v3-core-runtime.md`](../../../docs/architecture/v3-core-runtime.md) |
| Understand security, claims, and governance | [`../../../docs/architecture/v3-security-governance.md`](../../../docs/architecture/v3-security-governance.md) |
| Understand plugins and shipped extension families | [`../../../docs/architecture/v3-plugin-ecosystem.md`](../../../docs/architecture/v3-plugin-ecosystem.md) |
| Understand legacy v2 and migration context | [`../../../docs/architecture/v2-legacy-architecture.md`](../../../docs/architecture/v2-legacy-architecture.md), [`../../../docs/architecture/version-evolution-and-migration.md`](../../../docs/architecture/version-evolution-and-migration.md) |

## Key Entry Points
| Path | Role |
|---|---|
| [`../../../docs/architecture/README.md`](../../../docs/architecture/README.md) | Master guide for the active architecture. |
| [`../../../docs/README.md`](../../../docs/README.md) | Repo-level docs landing page. |
| [`../README.md`](../README.md) | V3 bridge index that points here. |

## How It Works
Use this directory as a compatibility signpost, not as a second documentation system. Pick the question you have, then follow the matching link into the repo-level architecture corpus.

## Why It Is Designed This Way
The old V3 README expected a guides directory. Restoring the path keeps existing links stable while consolidating current truth into one fresh documentation corpus.

## Dependencies
This page depends only on the repo-level architecture pages under `../../../docs/architecture/`.

## Operational/Test Notes
If a new guide is needed, add it to the repo-level corpus first and then link it here if the `v3/docs/guides/` bridge should expose it.

## Known Drift
The historical V3 docs layout implied a larger guide tree than this checkout actually contained. This bridge intentionally points to the new architecture corpus instead of recreating a second full guide set under `v3/`.

## Related Docs
| Document | Why it matters |
|---|---|
| [`../README.md`](../README.md) | V3 docs bridge index. |
| [`../../implementation/adrs/README.md`](../../implementation/adrs/README.md) | ADR detail once you move from guide-level understanding to implementation history. |
