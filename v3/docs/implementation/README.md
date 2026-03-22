# V3 Implementation Bridge
Last updated: 2026-03-21

## Purpose
This page provides the missing `v3/docs/implementation/` landing page and routes readers to the implementation-level material that already exists in the repository.

## Status
| Surface | Status | Notes |
|---|---|---|
| This bridge page | `supporting asset` | Compatibility layer for `v3/README.md`. |
| `v3/implementation/adrs/` | `supporting asset` | Canonical in-tree ADR collection. |
| `v3/docs/adr/` and `v3/docs/ddd/` | `supporting asset` | Existing local implementation notes and domain-model material. |
| Repo-level architecture corpus | `active` | Current code-backed architecture story and drift notes. |

## Covered Areas
| Implementation topic | Recommended page |
|---|---|
| ADR baseline and status | [`../../implementation/adrs/README.md`](../../implementation/adrs/README.md), [`../../implementation/adrs/ADR-STATUS-SUMMARY.md`](../../implementation/adrs/ADR-STATUS-SUMMARY.md) |
| Security review and audit trail | [`../../implementation/adrs/SECURITY-REVIEW-SUMMARY.md`](../../implementation/adrs/SECURITY-REVIEW-SUMMARY.md), [`../../implementation/adrs/ADR-061-deep-audit-findings-2026-03.md`](../../implementation/adrs/ADR-061-deep-audit-findings-2026-03.md) |
| Existing V3 ADR notes | [`../adr/README.md`](../adr/README.md) |
| Existing DDD notes | [`../ddd/coherence-engine/README.md`](../ddd/coherence-engine/README.md), [`../ddd/quality-engineering/README.md`](../ddd/quality-engineering/README.md) |
| Current runtime ownership and drift | [`../../../docs/architecture/v3-core-runtime.md`](../../../docs/architecture/v3-core-runtime.md), [`../../../docs/architecture/v3-surfaces-and-operations.md`](../../../docs/architecture/v3-surfaces-and-operations.md), [`../../../docs/architecture/v3-security-governance.md`](../../../docs/architecture/v3-security-governance.md), [`../../../docs/architecture/v3-plugin-ecosystem.md`](../../../docs/architecture/v3-plugin-ecosystem.md) |

## Key Entry Points
| Path | Role |
|---|---|
| [`../../implementation/adrs/README.md`](../../implementation/adrs/README.md) | ADR collection entrypoint. |
| [`../adr/README.md`](../adr/README.md) | Existing V3 docs ADR landing page. |
| [`../../../docs/architecture/README.md`](../../../docs/architecture/README.md) | Current architecture narrative. |

## How It Works
This directory bridges from the historical V3 docs tree into three implementation-detail sources:

1. ADRs under `v3/implementation/adrs/`
2. Existing V3-local ADR and DDD pages under `v3/docs/`
3. Fresh repo-level architecture notes that explain what the implementation currently does and where historical documentation drifts

## Why It Is Designed This Way
Implementation detail is already split across ADRs, DDD notes, and package-level architecture pages. This bridge keeps the old link target alive without duplicating that material into a new parallel tree.

## Dependencies
| Dependency | Notes |
|---|---|
| `v3/implementation/adrs/` | Main design-history source. |
| `v3/docs/adr/` and `v3/docs/ddd/` | Existing V3-local implementation notes. |
| `../../../docs/architecture/` | Current architecture truth and drift callouts. |

## Operational/Test Notes
When implementation changes, update the repo-level architecture pages for current truth and update ADR/DDD notes only when the design history or local implementation notes themselves change.

## Known Drift
The implementation material is intentionally split. ADRs and DDD notes explain design intent and local history, while the repo-level architecture corpus explains the live checkout. Readers should use both when the code and older prose have diverged.

## Related Docs
| Document | Why it matters |
|---|---|
| [`../README.md`](../README.md) | V3 docs bridge index. |
| [`../guides/README.md`](../guides/README.md) | Goal-oriented bridge back into the fresh architecture corpus. |
