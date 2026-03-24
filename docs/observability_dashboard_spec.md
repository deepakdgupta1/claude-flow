# Observability Dashboard — Product Requirements Document (v0.1 Draft)

> **Status**: Draft — First brainstorm pass
> **Author**: Deepak (PM) + Claude (spec writer)
> **Date**: 2026-03-23
> **Source**: [User Journeys Brainstorm](./user_journeys_brainstorm.md) §8 (Deferred Items)
> **Audience**: Tech Leads, Engineering Managers

---

## 1. Problem Statement

The autonomous multi-agent orchestration platform runs complex swarms of 6-8+ agents across 12 distinct journey types — from greenfield bootstrapping to incident response. Today, **there is zero visibility** into what's happening inside a running swarm or how past swarms performed. This is the equivalent of launching a fleet of autonomous drones with no radar screen and no flight recorder.

**Who feels this**: Tech Leads and Engineering Managers who are responsible for the quality, cost, and reliability of agent-driven software development. They need to answer questions like:

- "Why did that refactoring swarm take 45 minutes and burn $12 in tokens?"
- "Which agent keeps failing during code review journeys?"
- "Is our memory system actually helping, or are agents re-fetching the same context repeatedly?"
- "Are we getting better over time, or are quality scores plateauing?"

**Cost of not solving this**: Without observability, the platform is a black box. Users can't diagnose failures, can't optimize costs, can't validate that the learning/improvement mechanisms (§7 of brainstorm) are actually working, and can't build trust that the platform is doing the right thing autonomously. This is the #1 blocker to confident adoption beyond early experimentation.

---

## 2. Goals

| # | Goal | Measure |
|---|------|---------|
| G1 | **Real-time situational awareness** — Tech leads can see what every agent in a running swarm is doing, right now | Time-to-diagnose a stuck/failed agent drops from "unknown" to < 60 seconds |
| G2 | **Cost transparency** — Token spend is fully attributable by agent role, journey type, and phase | 100% of token spend is categorized; no "unattributed" bucket |
| G3 | **Quality trend tracking** — Outcomes and quality metrics are tracked over time, enabling data-driven improvement | Users can answer "are we getting better?" with a chart, not a gut feeling |
| G4 | **Memory system health** — Visibility into memory tier utilization, retrieval quality, and context window efficiency | Context window waste (loaded-but-unused context) is measurable and reducible |
| G5 | **Failure diagnosis** — When something goes wrong, the dashboard provides enough information to understand why without digging through raw logs | 80% of failures are diagnosable from the dashboard alone (no log spelunking) |

---

## 3. Non-Goals

| # | Non-Goal | Rationale |
|---|----------|-----------|
| NG1 | **Alerting / PagerDuty integration** | The platform's Error Handler Agent (§5 of brainstorm) already handles real-time error routing. The dashboard is for human understanding, not automated alerting. Alerting is a separate initiative. |
| NG2 | **Multi-tenant / team-based views** | Per brainstorm §8, multi-tenant boundaries are deferred. This dashboard is for a single user's projects. |
| NG3 | **Agent configuration / tuning from the dashboard** | This is a read-only observability tool. Configuration and tuning happen through the CLI and config files. Adding write operations introduces risk and complexity. |
| NG4 | **Raw log viewer / full log search** | The dashboard provides structured, curated views. For raw log deep-dives, users use existing log tooling. The dashboard should make raw logs unnecessary for 80% of cases. |
| NG5 | **Billing / invoicing features** | Token cost tracking is for optimization insights, not for generating invoices or managing payment. |

---

## 4. User Stories

### Tech Lead / Architect

- **As a tech lead**, I want to see a live topology visualization of my running swarm so that I can understand the current agent structure, who's doing what, and where bottlenecks are forming.
- **As a tech lead**, I want to see which agents have errored, retried, or been reassigned during a journey so that I can identify reliability issues with specific agent roles or model choices.
- **As a tech lead**, I want to compare the token cost of similar journeys over time so that I can validate that the platform's token optimization strategies (§3.1 — deterministic automation, context bundles, etc.) are actually reducing spend.
- **As a tech lead**, I want to see which memory tier served each context retrieval and whether the retrieved context was actually used so that I can assess if the memory system is efficient or wasteful.
- **As a tech lead**, I want to review the peer review scores and reputation scores of agents across journeys so that I can identify underperforming agent types and decide whether to adjust strategy or model routing.

### Engineering Manager

- **As an engineering manager**, I want a high-level summary of all swarm runs this week — success rate, total cost, average duration — so that I can report on platform ROI and efficiency without diving into individual runs.
- **As an engineering manager**, I want to see human override frequency and the logged rationale for overrides so that I can assess whether the platform's autonomous decisions are trustworthy and improving.
- **As an engineering manager**, I want to see quality trends over time (journey success rate, bug escape rate, review scores) so that I can confirm the platform's learning mechanisms (§7) are delivering measurable improvement.
- **As an engineering manager**, I want a cost breakdown by journey type (feature dev vs. bug fix vs. refactor, etc.) so that I can understand where tokens are being spent and whether the distribution is reasonable.
- **As an engineering manager**, I want to see the deterministic automation conversion rate — how many tasks have been converted from agent-driven to script-driven — so that I can track the platform's self-optimization progress.

### Solo Developer (secondary persona)

- **As a solo developer**, I want a quick "health check" view of my current swarm so that I can glance at it and know if things are on track without reading detailed metrics.
- **As a solo developer**, I want to understand why a journey failed or produced low-quality output so that I can decide whether to retry, adjust parameters, or investigate manually.

---

## 5. Requirements

### 5.1 Must-Have (P0) — MVP

#### P0-1: Live Swarm Topology View

A real-time visualization of the active swarm's topology (hierarchical, mesh, federated, etc.) showing each agent as a node with its current state.

**Acceptance Criteria:**
- [ ] Displays the swarm topology as a graph with agents as nodes and communication channels as edges
- [ ] Each agent node shows: agent role, current state (idle / working / waiting / errored / handed-over), model/runtime (e.g., Claude, Codex), and current task summary
- [ ] Topology updates in real-time (< 5 second refresh) as agents change state
- [ ] Clicking an agent node opens a detail panel with: task history, token usage, error log, and messages sent/received
- [ ] Failed/errored agents are visually highlighted (red border or similar)
- [ ] Supports all topology types from brainstorm §3.5: pipeline, broadcast, mesh, adversarial, hierarchical, federated, concierge

#### P0-2: Token & Cost Tracking Panel

Real-time and historical view of token consumption, attributable by agent, journey, and phase.

**Acceptance Criteria:**
- [ ] Shows cumulative token spend for the current session, broken down by agent role
- [ ] Shows burn rate (tokens/minute) for the active swarm with a trend line
- [ ] Historical view: cost per journey, filterable by journey type (Journey 1-12 from brainstorm)
- [ ] Each journey's cost is broken down by phase (e.g., for Journey 2: Spec Refinement → Design → Implementation → Testing → Review → Integration)
- [ ] Budget utilization bar showing spend vs. configured quota (session / daily / monthly)
- [ ] Token optimization indicator: shows which optimization strategies (§3.1) fired and estimated savings

#### P0-3: Journey Outcome Summary

A summary view of completed journeys with pass/fail status, duration, cost, and quality indicators.

**Acceptance Criteria:**
- [ ] Lists all completed journeys with: journey type, trigger, duration, total token cost, outcome (success / partial / failed), and human checkpoint count
- [ ] Filterable by date range, journey type, and outcome status
- [ ] Clicking a journey opens a detail view with the full agent activity timeline
- [ ] Shows the Error Handler Agent's actions for failed journeys: root cause category, routing decision (retry / reassign / workaround / escalate)

#### P0-4: Agent Error & Recovery Log

Structured view of all errors, retries, reassignments, and escalations across agents.

**Acceptance Criteria:**
- [ ] Lists all error events with: timestamp, agent role, error category (model error, tool error, coordination error, resource error — per brainstorm §5), severity, and resolution action
- [ ] Shows the Error Handler Agent's full decision chain for each error (detection → RCA → routing decision → outcome)
- [ ] Filterable by error category, agent role, and severity
- [ ] Aggregated view: error rate by agent role over time (trend chart)

---

### 5.2 Nice-to-Have (P1) — Fast Follow

#### P1-1: Memory System Health Panel

Visibility into the 4-tier memory system's utilization and retrieval quality.

**Acceptance Criteria:**
- [ ] Shows memory tier distribution: how much data is in hot (working) / warm (project) / cold (organizational) memory
- [ ] Context window utilization per agent: what % of each agent's context window is in use, and what % of loaded context was actually referenced in the agent's output
- [ ] RAG retrieval quality: for each memory retrieval, shows the query, retrieved items, relevance scores, and whether the retrieved context was used
- [ ] Memory compression events: when working memory was compressed to session memory, with before/after token counts
- [ ] Handover events: when the Budget Agent triggered a context handover, with the handover document size and fresh agent's validation status

#### P1-2: Quality & Learning Trends Dashboard

Historical view of quality metrics that tracks the platform's learning and improvement over time.

**Acceptance Criteria:**
- [ ] Journey success rate trend (weekly / monthly) — overall and by journey type
- [ ] Peer review scores over time (from brainstorm §3.3 — Critic Agent evaluations)
- [ ] Agent reputation scores over time, with the ability to see which agents are trending up/down
- [ ] Human override frequency trend — are overrides decreasing as the platform learns?
- [ ] Deterministic automation conversion rate — how many task patterns have been converted to scripts (per brainstorm §9.3, threshold of 3 repetitions)
- [ ] Topology performance tracking — which topologies are selected for which task types, and their success rates

#### P1-3: Human Override Log

Dedicated view of all human override decisions with logged rationale.

**Acceptance Criteria:**
- [ ] Lists all human override events with: timestamp, checkpoint type, original agent recommendation, human decision, and human rationale
- [ ] Filterable by journey type and checkpoint type
- [ ] Pattern detection: highlights repeated override patterns that might indicate the platform needs adjustment (surfaced for human review per brainstorm §9.4 — no auto-learning)

#### P1-4: Cross-Runtime Agent Comparison

For swarms using cross-runtime binding (§3.2), compare performance across different AI runtimes.

**Acceptance Criteria:**
- [ ] Side-by-side comparison of agent performance by runtime (Claude vs. Codex vs. local models)
- [ ] Metrics: task completion time, token cost, error rate, quality scores
- [ ] Filterable by agent role and task type

---

### 5.3 Future Considerations (P2) — Design For, Don't Build

#### P2-1: Anomaly Detection & Smart Alerts
The dashboard should be architected to support future anomaly detection (behavioral anomaly detection per brainstorm §3.3) and proactive alerting when agent behavior deviates from baselines.

#### P2-2: Comparative Journey Replay
Ability to replay two similar journeys side-by-side to understand why one succeeded and the other failed — like a "diff" for swarm executions.

#### P2-3: Custom Dashboard Builder
Let users create custom views by composing widgets/panels for their specific workflow.

#### P2-4: Multi-Project Aggregate View
When multi-tenant/multi-team support is added (brainstorm §8), the dashboard should support aggregate views across projects.

#### P2-5: Export & Reporting
Export dashboard data as PDF/CSV for stakeholder reporting and compliance audit trails (ties to Journey 12 — Compliance Auditing).

---

## 6. Success Metrics

### Leading Indicators (Days to Weeks Post-Launch)

| Metric | Target | Stretch | Measurement |
|--------|--------|---------|-------------|
| Dashboard adoption rate | 70% of active platform users visit dashboard weekly | 90% | Usage analytics |
| Time-to-diagnose agent failure | < 60 seconds from dashboard alone | < 30 seconds | User testing / dogfooding |
| Token cost attribution coverage | 100% of spend categorized by agent + journey | — | Automated audit of unattributed tokens |
| Dashboard page load time | < 3 seconds for real-time view | < 1 second | Performance monitoring |

### Lagging Indicators (Weeks to Months)

| Metric | Target | Stretch | Measurement |
|--------|--------|---------|-------------|
| Token cost reduction (driven by dashboard insights) | 15% reduction in avg. token cost per journey within 60 days | 25% | Historical cost comparison |
| Journey success rate improvement | 10% improvement within 90 days (users fix issues they can now see) | 20% | Journey outcome tracking |
| Reduction in "raw log spelunking" | 80% of failure investigations completed without leaving dashboard | 95% | User survey / dogfooding |
| Human override frequency decrease | 10% decrease in overrides within 90 days (indicates platform is learning and users trust it more) | 20% | Override log analysis |

---

## 7. Open Questions

| # | Question | Owner | Blocking? |
|---|----------|-------|-----------|
| Q1 | **Data retention**: How long do we retain granular agent-level telemetry (per-message token counts, per-retrieval relevance scores)? The memory system has hot/warm/cold tiers — should dashboard telemetry follow the same pattern? | Engineering + Deepak | Non-blocking (can start with "keep everything" and optimize later) |
| Q2 | **Real-time transport**: What's the mechanism for streaming live agent state to the dashboard? WebSocket from the daemon? Polling the MCP? Event bus? | Engineering | Blocking for P0-1 (live topology view) |
| Q3 | **Token cost calculation**: How do we attribute token cost accurately when agents use different models/runtimes with different pricing? Do we use a canonical "token unit" or actual dollar cost? | Engineering + Deepak | Non-blocking (start with actual dollar cost, normalize later) |
| Q4 | **Dashboard hosting**: Is this a local web UI served by the daemon, a CLI-based TUI, or a hosted web app? Given the platform is for personal use (brainstorm §8), a local web UI seems right, but this affects architecture significantly. | Deepak | Blocking |
| Q5 | **Memory retrieval quality scoring**: The P1-1 spec calls for "relevance scores" on RAG retrievals. Does the memory system currently expose these, or does this require new instrumentation? | Engineering | Blocking for P1-1 |
| Q6 | **Agent reputation scoring**: The brainstorm (§3.3) mentions reputation scoring, but the current implementation status is unclear. Is this already instrumented, or does the dashboard need to compute it from raw outcome data? | Engineering | Blocking for P1-2 |
| Q7 | **Resident agent tracking**: Resident agents (§9.2 — Architect, Standards, Context agents) persist across journeys. How do we visualize their continuous state vs. transient agents that spin up and die? | Design + Engineering | Non-blocking |

---

## 8. Timeline Considerations

| Factor | Detail |
|--------|--------|
| **Hard dependencies** | P0-1 (live topology) depends on answering Q2 (real-time transport). P0-2 (token tracking) depends on the Budget Agent (§6) exposing structured cost data. |
| **Suggested phasing** | **Phase 1**: P0-1 through P0-4 (real-time ops + journey summaries). **Phase 2**: P1-1 through P1-4 (memory health, quality trends, overrides, cross-runtime). **Phase 3**: P2 items as the platform matures. |
| **No hard deadline** | This is a platform capability, not a customer commitment. Quality > speed per brainstorm principles. |
| **Instrumentation work** | Several P0 and P1 features require new telemetry events from agents, the Error Handler, the Budget Agent, and the memory system. This instrumentation work should be scoped separately and may be the longest lead-time item. |

---

## 9. Appendix: Data Model Sketch

The dashboard needs these core data entities (not a schema — just the conceptual model):

**Journey Run** — a single execution of a journey (type, trigger, start/end time, outcome, total cost, topology used)

**Agent Instance** — a single agent within a journey run (role, model/runtime, lifecycle events, token usage, error events, messages sent/received)

**Error Event** — an error occurrence (agent, timestamp, category, severity, Error Handler decision chain, resolution outcome)

**Memory Event** — a memory operation (retrieval query, tier, relevance score, context used/unused, compression event, handover event)

**Human Checkpoint** — a human decision point (checkpoint type, agent recommendation, human decision, rationale, timestamp)

**Token Ledger Entry** — a granular token usage record (agent, operation type, input/output tokens, model, estimated cost)

**Quality Score** — a quality evaluation (journey, evaluator agent, dimension, score, evidence)

---

*This is a first-draft spec for brainstorming purposes. All requirements, metrics, and priorities are open for discussion.*
