# V3 Security Governance
Last updated: 2026-03-21

## Purpose
This document captures the security governance boundary for V3: the core security primitives, the AI-defense layer, the claims/authority system, and the adjacent CLI, MCP, browser, and hook surfaces that can accept or execute untrusted input.

## Status
- `@claude-flow/security`: active
- `@claude-flow/claims`: active
- `@claude-flow/aidefence`: supporting asset
- Browser security integration: supporting asset
- Claims CLI wrapper: legacy
- ADR-022 integration plan: historical/unverified
- ADR-061, ADR-063, ADR-064, ADR-065: historical audit/remediation trail

## Covered Areas
- Core validation and execution controls in `@claude-flow/security`: password hashing, credential generation, token signing, Zod input validation, path validation, and allowlisted command execution.
- AI manipulation detection in `@claude-flow/aidefence`: prompt-injection detection, PII detection, learned mitigation, and optional vector-backed similarity search.
- Claim ownership and work coordination in `@claude-flow/claims`: claim/release/handoff, work stealing, load balancing, and contest windows.
- Browser navigation and data-entry safety: URL screening, HTTPS preference, blocked-domain checks, and PII masking.
- Execution-control surfaces in CLI, MCP, and hooks where input may cross a trust boundary.

## Key Entry Points
- [Security facade](../../v3/@claude-flow/security/src/index.ts)
- [Security domain service](../../v3/@claude-flow/security/src/domain/services/security-domain-service.ts)
- [Security application service](../../v3/@claude-flow/security/src/application/services/security-application-service.ts)
- [Input validator](../../v3/@claude-flow/security/src/input-validator.ts)
- [Path validator](../../v3/@claude-flow/security/src/path-validator.ts)
- [Safe executor](../../v3/@claude-flow/security/src/safe-executor.ts)
- [Token generator](../../v3/@claude-flow/security/src/token-generator.ts)
- [Claims facade](../../v3/@claude-flow/claims/src/index.ts)
- [Claim service](../../v3/@claude-flow/claims/src/application/claim-service.ts)
- [Work stealing service](../../v3/@claude-flow/claims/src/application/work-stealing-service.ts)
- [Claims MCP tools](../../v3/@claude-flow/claims/src/api/mcp-tools.ts)
- [Claims CLI commands](../../v3/@claude-flow/claims/src/api/cli-commands.ts)
- [AIDefence facade](../../v3/@claude-flow/aidefence/src/index.ts)
- [Threat detection service](../../v3/@claude-flow/aidefence/src/domain/services/threat-detection-service.ts)
- [Threat learning service](../../v3/@claude-flow/aidefence/src/domain/services/threat-learning-service.ts)
- [Browser facade](../../v3/@claude-flow/browser/src/index.ts)
- [Browser service](../../v3/@claude-flow/browser/src/application/browser-service.ts)
- [Browser security integration](../../v3/@claude-flow/browser/src/infrastructure/security-integration.ts)
- [Browser MCP tools](../../v3/@claude-flow/browser/src/mcp-tools/browser-tools.ts)
- [Browser hooks](../../v3/@claude-flow/browser/src/infrastructure/hooks-integration.ts)
- [ADR-012](../../v3/implementation/adrs/ADR-012-mcp-security-features.md), [ADR-013](../../v3/implementation/adrs/ADR-013-core-security-module.md), [ADR-016](../../v3/implementation/adrs/ADR-016-collaborative-issue-claims.md), [ADR-022](../../v3/implementation/adrs/ADR-022-aidefence-integration.md), [ADR-061](../../v3/implementation/adrs/ADR-061-deep-audit-findings-2026-03.md), [Security review summary](../../v3/implementation/adrs/SECURITY-REVIEW-SUMMARY.md)

## How It Works
`@claude-flow/security` is the hard control plane. `SecurityApplicationService` binds a `SecurityContext` to a principal, and `SecurityDomainService` checks whether a path or command is allowed before execution. `PathValidator` and `SafeExecutor` are the strongest guards; `InputValidator` and `TokenGenerator` provide supporting validation and signing.

`@claude-flow/claims` is the ownership plane. `ClaimService` enforces who owns an issue, who can release it, whether a handoff is pending, and whether a claim can be stolen or contested. The MCP layer parses with Zod, but the authoritative checks live in the service layer and repository state.

`@claude-flow/aidefence` is the adaptive detection plane. `ThreatDetectionService` flags prompt injection and PII, while `ThreatLearningService` stores learned patterns and mitigation outcomes. Learning is optional, and persistence only exists when a vector store is supplied.

`@claude-flow/browser` is the session-level navigation plane. `BrowserService.open()` consults `BrowserSecurityScanner` before navigation, and `fill()` can mask PII. The raw browser MCP tools do not automatically invoke that scanner, so URL safety is only enforced when callers choose the service path.

## Why It Is Designed This Way
The architecture separates deterministic enforcement from adaptive detection. Security primitives are kept narrow and synchronous where possible; claims ownership is enforced where the state lives; browser safety is advisory unless routed through the service; and AIDefence is treated as a supporting layer rather than a replacement for hard validation.

This reduces blast radius. A bad URL, command, or issue handoff should fail at the closest boundary, with the smallest trusted surface, instead of depending on a global policy layer.

## Dependencies
- `@claude-flow/security`: `bcrypt`, `zod`, and Node `crypto`
- `@claude-flow/aidefence`: optional `agentdb` for vector search; otherwise in-memory learning
- `@claude-flow/claims`: `zod` and Node `crypto`, with optional `@claude-flow/shared`
- Browser security path: `zod`, browser service adapters, and the adjacent memory/ReasoningBank integrations

## Operational/Test Notes
- Validate the service layer, not just the CLI or MCP wrapper.
- `security`: run the package tests and typecheck, then verify `auditSecurityConfig()` warnings for your deployment config.
- `claims`: test claim ownership, handoff acceptance, steal/contest windows, and rebalance behavior through both service and tool layers.
- `browser`: if URL safety matters, use `BrowserService.open()` or `isUrlSafe()`; do not assume the raw `browser/open` tool enforces scanning.
- `aidefence`: test both detection-only and learning-enabled modes, and verify the vector store path if persistence matters.

## Known Drift
- `browser/src/mcp-tools/browser-tools.ts` bypasses `BrowserSecurityScanner`; enforcement only happens when callers use `BrowserService`.
- `preBrowseHook` warns on non-HTTPS URLs, but it does not block them. `UrlSchema` also allows HTTP while `HttpsUrlSchema` exists separately.
- `claims/src/api/cli-commands.ts` still targets older MCP names such as `claims/claim` and `claims/list`, while `claims/src/api/mcp-tools.ts` registers `claims/issue_*` tools. Treat that as legacy alias drift.
- `claims` status vocabularies differ across layers (`pending_handoff`, `in_review`, `expired` in the service layer versus `review-requested`, `handoff-pending`, and similar CLI terms). Integrations need explicit mapping.
- ADR-022 describes an external `aidefence` package and richer integration surfaces than the current in-repo `@claude-flow/aidefence` facade. Use it as historical context, not current runtime truth.
- The AIDefence README and ADRs describe 50+ patterns and production AIMDS integration, but the checked-in service is a smaller in-repo detector with optional learning storage.
- `PathSchema` and `UrlSchema` are lightweight guards only; they do not replace `PathValidator`, `SafeExecutor`, or browser service-level checks.
- ADR-061/063/064/065 are valuable audit records, but most of their findings sit in adjacent CLI, MCP, plugin, or transfer surfaces rather than this security core.

## Related Docs
- [ADR-012: MCP Security Features](../../v3/implementation/adrs/ADR-012-mcp-security-features.md)
- [ADR-013: Core Security Module](../../v3/implementation/adrs/ADR-013-core-security-module.md)
- [ADR-016: Collaborative Issue Claims](../../v3/implementation/adrs/ADR-016-collaborative-issue-claims.md)
- [ADR-022: AIDefence Integration](../../v3/implementation/adrs/ADR-022-aidefence-integration.md)
- [ADR-061: Deep System Audit Findings](../../v3/implementation/adrs/ADR-061-deep-audit-findings-2026-03.md)
- [Security Review Summary](../../v3/implementation/adrs/SECURITY-REVIEW-SUMMARY.md)
