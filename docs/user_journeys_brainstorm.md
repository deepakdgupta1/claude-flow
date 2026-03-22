# User Journeys: Autonomous Multi-Agent Orchestration Platform for Software Development

> **Context**: This document brainstorms the end-to-end user journeys that emerge when software development is conducted through a general-purpose autonomous multi-agent orchestration platform. The platform provides automated topology selection, multi-agent coordination, shared multi-tier memory, SOTA context management, continuous token optimization, and cross-runtime agent binding.

---

## Platform Capabilities

| Capability | Description |
|---|---|
| **Automated Topology Selection** | Dynamically selects agent swarm topology (pipeline, DAG, hierarchical, mesh, broadcast, etc.) based on task characteristics |
| **Multi-Agent Coordination** | Agents negotiate, delegate, merge, and resolve conflicts autonomously |
| **Shared Multi-Tier Memory** | Working memory → session memory → project memory → organizational memory |
| **Context Management** | Shared context across agents + individual per-agent context; intelligent compression and retrieval |
| **Continuous Token Usage Optimization** | Actively identifies and exploits opportunities to reduce token consumption (see §3.1) |
| **Cross-Runtime Agent Binding** | Binds different AI Agent runtimes to a single session for seamless inter-agent communication regardless of underlying runtime |
| **Quality-First Execution** | All agent work is optimized for thoroughness and highest quality; speed is secondary |

---

## 1. Personas

| Persona | Role | Primary Platform Interaction |
|---|---|---|
| **Solo Developer** | Individual contributor | Spawns agent swarms to amplify personal output |
| **Tech Lead / Architect** | System designer, standards enforcer | Defines guardrails, reviews agent-generated architectures |
| **Product Manager** | Requirements owner | Feeds intent; reviews generated specs and progress |
| **QA / SDET** | Quality gatekeeper | Configures & reviews testing agent strategies |
| **DevOps / Platform Engineer** | Infrastructure & CI/CD owner | Orchestrates deployment & infra agents |
| **Engineering Manager** | Team coordinator | Monitors swarm performance, cost, and velocity |
| **New Hire / Onboardee** | Learning the codebase | Uses agents as interactive guides and pair programmers |

---

## 2. User Journeys

### Journey 1 — Greenfield Project Bootstrapping

**Trigger**: User describes a new project idea or provides a PRD/brief.

**Flow**:
1. **Intent Capture** — User provides high-level description (natural language, PRD, sketch, reference URLs).
2. **Topology Selection** — Platform selects a **hierarchical fan-out** topology: an Architect Agent at the top, with specialized sub-agents (scaffolding, design system, data model, infra-as-code, CI/CD).
3. **Decomposition & Planning** — Architect Agent breaks the project into modules, sets up a task graph (DAG) with dependencies.
4. **Parallel Execution** — Sub-agents execute in parallel where the DAG allows — scaffolding the repo, generating schema, setting up pipelines, creating initial components.
5. **Memory Seeding** — Shared project memory is seeded with architectural decisions, tech stack choices, coding conventions, and a living ADR log.
6. **Human Checkpoints** — Platform surfaces high-impact decision points for human review (tech stack trade-offs, auth strategy, deployment targets). Work continues in parallel on non-blocked items.
7. **Output** — Running project with CI/CD, initial test suite, documentation, and a populated project memory for all future agents.

**Key Agent Roles**: Architect, Scaffolder, Schema Designer, CI/CD Agent, Documentation Agent.

---

### Journey 2 — End-to-End Feature Development

**Trigger**: Feature request (Jira ticket, natural language, or spec document).

**Flow**:
1. **Context Retrieval** — The platform pulls relevant context from project memory: architecture, existing patterns, related modules, prior decisions.
2. **Topology Selection** — **Pipeline with parallel branches**: Spec Refinement → Design → Implementation (parallel across services/layers) → Testing → Review → Integration.
3. **Spec Refinement** — A Spec Agent clarifies ambiguities, generates acceptance criteria, and identifies affected components.
4. **Design** — Design Agent proposes the approach, including API contracts, data model changes, and UI wireframes (if applicable). Surfaces trade-offs for human approval.
5. **Implementation** — Multiple Coder Agents work in parallel across layers (frontend, backend, data, infra). Shared context ensures API contracts are honored; agents negotiate on interface boundaries.
6. **Testing** — Test Agents generate and execute unit, integration, and e2e tests. Coverage gaps are identified and filled.
7. **Code Review** — Reviewer Agents check for consistency with project standards, security, performance, and correctness. Findings are compiled and optionally escalated to human.
8. **Integration** — PR is assembled, CI passes, and changeset is ready for merge.

**Memory Usage**: Session memory retains the feature context; project memory is updated with new patterns and decisions.

---

### Journey 3 — Complex Bug Investigation & Resolution

**Trigger**: Bug report, error log, alert, or user-reported issue.

**Flow**:
1. **Triage** — A Triage Agent ingests the bug report, pulls error logs, stack traces, and related telemetry from organizational memory.
2. **Topology Selection** — **Mesh/collaborative** topology for exploratory investigation; agents communicate laterally to share hypotheses.
3. **Hypothesis Generation** — Multiple Investigation Agents explore different angles in parallel: recent commits, dependency changes, data anomalies, infrastructure drift.
4. **Evidence Gathering** — Agents query logs, trace requests, inspect state, and reproduce the issue in a sandboxed environment.
5. **Root Cause Convergence** — Agents share findings through shared working memory, debating and narrowing hypotheses. The platform detects convergence and elevates the strongest hypothesis.
6. **Fix Proposal** — A Fix Agent generates a patch, with tests that reproduce the bug (regression tests) and validate the fix.
7. **Verification** — The fix is tested in the sandbox. Memory is updated with the incident post-mortem.

**Key Differentiator**: Mesh topology allows agents to pivot dynamically; shared memory prevents duplicate investigation paths.

---

### Journey 4 — Large-Scale Refactoring & Technical Debt Reduction

**Trigger**: Tech lead identifies a cross-cutting refactor (e.g., migrate from REST to gRPC, extract a shared library, upgrade a framework version).

**Flow**:
1. **Impact Analysis** — An Analysis Agent scans the entire codebase to map all affected files, dependencies, and downstream impacts.
2. **Topology Selection** — **Hierarchical with DAG constraints**: Coordinator Agent plans the migration order based on dependency graph.
3. **Migration Plan** — The Coordinator generates a phased migration plan with rollback points.
4. **Batch Execution** — Worker Agents process files/modules in dependency order. Each batch is validated before the next begins.
5. **Cross-Agent Consistency** — Shared context ensures naming conventions, patterns, and interfaces remain consistent across all changes.
6. **Incremental Verification** — After each batch, test agents validate that existing tests pass and new patterns are correctly applied.
7. **Human Gating** — Critical phases (breaking API changes, database migrations, **file deletions**) require explicit human sign-off before proceeding. Work continues in parallel on non-blocked items.

**Memory Usage**: Organizational memory retains refactoring patterns for reuse across projects.

---

### Journey 5 — Automated Code Review & Quality Assurance

**Trigger**: PR submitted (by human or agent).

**Flow**:
1. **Context Loading** — The platform loads project conventions, past review comments, known anti-patterns, and the PR's related feature context from memory.
2. **Topology Selection** — **Parallel broadcast**: Multiple specialized review agents run concurrently.
3. **Multi-Dimensional Review**:
   - **Correctness Agent** — Logical errors, edge cases, null safety.
   - **Security Agent** — Vulnerability scanning, secrets detection, input validation.
   - **Performance Agent** — Algorithmic complexity, N+1 queries, memory leaks.
   - **Style Agent** — Coding standards, naming conventions, documentation completeness.
   - **Architecture Agent** — Conformance with system design, appropriate abstractions.
4. **Synthesis** — A Review Coordinator aggregates findings, removes duplicates, prioritizes by severity, and generates a unified review.
5. **Auto-Fix Suggestions** — For deterministic issues (formatting, simple bugs), agents propose in-line fixes.
6. **Learning** — Review outcomes are fed back into project memory to improve future reviews.

---

### Journey 6 — System Design & Architecture Planning

**Trigger**: New system/service/module needs to be designed, or a major redesign is needed.

**Flow**:
1. **Requirements Ingestion** — Platform ingests requirements, constraints, NFRs, existing architecture docs from organizational memory.
2. **Topology Selection** — **Debate/adversarial** topology: Multiple Design Agents propose competing architectures.
3. **Parallel Design Proposals** — Each Design Agent generates a complete architecture proposal (diagrams, component breakdown, data flow, API contracts, deployment model).
4. **Adversarial Review** — Agents critique each other's proposals against NFRs: scalability, cost, operability, security, team expertise.
5. **Synthesis** — A Synthesis Agent merges the strongest elements from each proposal into a recommended design.
6. **Human Review** — The synthesized design is presented with trade-off analysis and visualizations for human decision-making.
7. **Decision Recording** — Selected architecture and rationale are stored as ADRs in project memory.

**Key Differentiator**: Adversarial topology surfaces blind spots that a single agent or human designer might miss.

---

### Journey 7 — Comprehensive Testing Strategy & Execution

**Trigger**: A new feature is ready for testing, or a periodic test health review is requested.

**Flow**:
1. **Test Gap Analysis** — A Test Analyst Agent reviews existing coverage maps, mutation testing results, and recent bug escape history from project memory.
2. **Topology Selection** — **DAG pipeline**: Strategy → Generation → Execution → Reporting.
3. **Strategy Design** — The analyst produces a test strategy: what to test, test types (unit, integration, contract, e2e, chaos, load), priority, and risk areas.
4. **Test Generation** — Specialized agents generate tests in parallel:
   - **Unit Test Agent** — Per-function/method tests with edge cases.
   - **Integration Test Agent** — Service interaction tests, database tests.
   - **E2E Test Agent** — User-flow tests with browser/API automation.
   - **Performance Test Agent** — Load profiles, benchmark scripts.
5. **Execution** — Tests run in parallel; results flow into shared working memory. Regression tests are automated via scripts with clear information-exchange contracts (see §3.1, Example 3).
6. **Intelligent Reporting** — A Reporting Agent surfaces failures, flaky tests, coverage trends, and risk assessment.

---

### Journey 8 — Incident Response & Production Debugging

**Trigger**: PagerDuty alert, anomaly detection, or user-reported production issue.

**Flow**:
1. **Alert Enrichment** — An Enrichment Agent pulls recent deployments, config changes, dependency status, and historical incidents from organizational memory.
2. **Topology Selection** — **Hierarchical with escalation**: Incident Commander Agent at top, with diagnostic sub-agents.
3. **Parallel Diagnostics**:
   - **Log Agent** — Scans and correlates logs across services.
   - **Metrics Agent** — Identifies anomalies in dashboards and time-series data.
   - **Trace Agent** — Follows distributed traces to find bottlenecks or failures.
   - **Infra Agent** — Checks infrastructure health, resource exhaustion, network issues.
4. **Mitigation** — Once root cause is identified, a Mitigation Agent proposes actions (rollback, feature flag, config change, hotfix).
5. **Communication** — A Comms Agent drafts status updates for stakeholders.
6. **Post-Mortem** — After resolution, a Post-Mortem Agent drafts the incident review document and updates organizational memory with learnings.

---

### Journey 9 — Developer Onboarding & Codebase Exploration

**Trigger**: New team member joins a project, or a developer needs to understand an unfamiliar area.

**Flow**:
1. **Profile Assessment** — The platform assesses the developer's experience level and what they need to learn.
2. **Topology Selection** — **Concierge/single-guide** with specialist backup agents: one primary Guide Agent, calling on specialists as needed.
3. **Guided Exploration** — The Guide Agent walks the developer through:
   - Architecture overview and key decisions (from project memory ADRs).
   - Codebase structure and key modules.
   - Development workflows, conventions, and tooling.
   - Recent changes and active feature areas.
4. **Interactive Q&A** — Developer asks questions; the Guide Agent retrieves answers from project memory or delegates to a Specialist Agent.
5. **Practice Tasks** — The platform suggests small, real tasks calibrated to the developer's level, providing real-time guidance and review.
6. **Memory Update** — Common onboarding questions are fed back into project memory to improve documentation and future onboarding.

---

### Journey 10 — Multi-Repository / Multi-Service Coordinated Changes

**Trigger**: A change that spans multiple repositories, services, or teams.

**Flow**:
1. **Dependency Mapping** — A Dependency Agent maps the blast radius across repos, services, and teams using organizational memory.
2. **Topology Selection** — **Federated hierarchical**: A Global Coordinator Agent, with per-repo Local Coordinator Agents, each commanding their own worker agents.
3. **Contract Negotiation** — Interface agents across repos negotiate API contracts, schema changes, and compatibility constraints via shared context.
4. **Staged Rollout Plan** — The Global Coordinator produces a rollout plan respecting inter-service dependencies and deployment order.
5. **Parallel Implementation** — Each repo's agent swarm works independently within agreed contracts, with shared memory ensuring consistency.
6. **Cross-Repo Integration Testing** — Integration agents spin up multi-service test environments to validate end-to-end flows.
7. **Coordinated Deployment** — Deployment agents across repos coordinate the release using the staged plan, with automatic rollback triggers.

---

### Journey 11 — Documentation & Knowledge Management

**Trigger**: Code changes merged, architecture decisions made, or periodic documentation refresh.

**Flow**:
1. **Change Detection** — A Watcher Agent monitors merged changes and identifies documentation gaps.
2. **Topology Selection** — **Pipeline**: Change Analysis → Draft → Review → Publish.
3. **Intelligent Drafting** — Doc Agents generate/update multiple doc types:
   - API reference docs (from code + contract definitions).
   - Architecture docs (from ADRs in project memory).
   - Runbooks (from incident history in organizational memory).
   - Onboarding guides (from onboarding journey interactions).
4. **Consistency Check** — A Consistency Agent verifies cross-references, detects stale content, and ensures terminology alignment.
5. **Human Review** — Generated docs are surfaced for human review with change diffs highlighted.
6. **Publishing** — Approved docs are published and indexed in project memory for future agent retrieval.

---

### Journey 12 — Continuous Compliance & Security Auditing

**Trigger**: Scheduled audit, regulatory change, or pre-release compliance check.

**Flow**:
1. **Policy Loading** — The platform loads compliance requirements (SOC2, GDPR, HIPAA, etc.) from organizational memory.
2. **Topology Selection** — **Parallel broadcast with aggregation**: Multiple audit agents run concurrently against different compliance dimensions.
3. **Automated Auditing**:
   - **Data Flow Agent** — Traces PII/sensitive data flows, checks encryption and retention.
   - **Access Control Agent** — Reviews IAM policies, service accounts, secrets management.
   - **Dependency Agent** — Scans for vulnerable or banned dependencies (license + CVE).
   - **Code Pattern Agent** — Checks for banned coding patterns (e.g., hard-coded credentials, insecure deserialization).
4. **Gap Analysis** — An Aggregator Agent compiles findings into a compliance gap report with severity and remediation priority.
5. **Auto-Remediation** — For deterministic fixes (dependency upgrades, config changes), agents propose and apply patches.
6. **Audit Trail** — All findings and remediations are logged in organizational memory for future audits.

---

## 3. Cross-Cutting Themes

### 3.1 Continuous Token Usage Optimization

The platform actively and continuously seeks to reduce token consumption without sacrificing quality. This is a **platform-level capability**, not an afterthought.

| Strategy | Description | Example |
|---|---|---|
| **Deterministic Automation of Repetitive Tasks** | Identifies tasks that agents perform repeatedly and converts them into deterministic scripts/code, so agents don't spend tokens reinventing the wheel each time. | Linting, formatting, boilerplate generation, standard file scaffolding — all handled by scripts after the first agent-driven execution proves the pattern. |
| **Specialist Context Bundle Serving** | Instead of letting agents fill their context window with entire files, the platform serves pre-curated, minimal context bundles containing only the relevant slices of the codebase. | An agent working on a function gets only that function, its callers, its tests, and relevant type definitions — not the entire file or repo. |
| **Script-Based Execution with Information Exchange Contracts** | Predictable, repetitive tasks (e.g., regression testing, build validation) are automated via scripts/tools that have a clear, concise contract for input/output, so the agent's context window isn't flooded with verbose tool output. | Regression test runner returns only a structured summary (pass/fail counts, failed test names, error snippets) rather than the full test output. |
| **Context Compression & Summarization** | Long-running sessions periodically compress working memory into concise summaries stored in higher memory tiers. | After completing a sub-task, the agent's detailed working memory is compressed into a 200-token summary in session memory. |
| **Incremental Context Loading** | Context is loaded on-demand and incrementally, not eagerly. Agents request context slices as needed. | An agent starts with a task description and architectural overview, then pulls in specific file contents only when it needs to edit them. |

### 3.2 Cross-Runtime Agent Binding

The platform supports binding different AI Agent runtimes (e.g., different LLM providers, local models, specialized fine-tuned models) to a single orchestration session. All agents communicate through the platform's shared memory and coordination layer, regardless of their underlying runtime.

**Key properties**:
- **Runtime-Agnostic Communication** — Agents exchange structured messages through the platform's message bus, not through runtime-specific APIs.
- **Heterogeneous Swarms** — A single swarm can include agents powered by different models (e.g., a GPT-based architect, a Claude-based coder, a local fine-tuned security scanner).
- **Seamless Context Sharing** — The platform's memory tiers abstract away runtime differences; all agents read from and write to the same shared memory.

### 3.3 Peer Review & Agent Improvement Mechanism

AI Agents within the swarm continuously improve through structured peer review. This draws from SOTA research on multi-agent verification.

| Mechanism | How It Works |
|---|---|
| **Critic Agent Role** | Dedicated Critic Agents evaluate the output of Worker Agents against quality criteria (correctness, completeness, adherence to standards). This mirrors the "Critic Agent" pattern from SOTA literature, where a specialized review agent drastically improves output quality. |
| **Reputation Scoring** | Agents maintain internal quality scores based on historical output evaluations. Higher-scoring agents are preferred for critical tasks; lower-scoring agents are flagged for strategy adjustment or replaced. |
| **Process Verification** | Intermediate reasoning steps (not just final outputs) are verified using process reward models. This catches flawed reasoning early, before it compounds. |
| **Adversarial Validation** | For high-stakes outputs (architecture decisions, security-critical code), a second agent independently attempts the same task; outputs are compared for consensus. Disagreements trigger deeper review. |
| **Behavioral Anomaly Detection** | The platform monitors agent behavior baselines (output patterns, tool usage, reasoning chains) and flags deviations that may indicate hallucination, drift, or degraded performance. |
| **Feedback-to-Memory Loop** | Peer review outcomes (approved corrections, rejected outputs, identified anti-patterns) are written to project memory, so all agents learn from each other's mistakes over time. |

### 3.4 Memory Tiers in Action

| Tier | Scope | Example Usage |
|---|---|---|
| **Working Memory** | Single agent execution | An agent's scratchpad during code generation |
| **Session Memory** | Single user session / task | Feature context during a multi-step development journey |
| **Project Memory** | Entire project lifetime | Architecture decisions, coding conventions, known issues |
| **Organizational Memory** | Across all projects | Compliance policies, incident patterns, reusable architecture patterns |

### 3.5 Topology Selection Patterns

| Task Shape | Recommended Topology | Example Journey |
|---|---|---|
| Linear, sequential steps | **Pipeline** | Documentation update |
| Independent parallel execution | **Broadcast / Fan-out** | Code review, compliance audit |
| Exploration with lateral sharing | **Mesh** | Bug investigation |
| Competing designs / perspectives | **Adversarial / Debate** | Architecture design |
| Multi-level decomposition | **Hierarchical** | Greenfield project, large refactor |
| Cross-team / cross-repo | **Federated** | Multi-repo coordinated changes |
| Interactive guidance | **Concierge** | Developer onboarding |

### 3.6 Context Management Patterns

- **Context Inheritance** — Child agents inherit relevant parent context slices, not the full dump.
- **Context Negotiation** — Agents at interface boundaries exchange contract-relevant context without exposing internals.
- **Context Compression** — Long-running tasks periodically compress working memory into summaries stored in session/project memory.
- **Context Recovery** — If an agent fails or is replaced, session memory allows seamless continuation.

---

## 4. Human-in-the-Loop Policy

Based on our discussion, the platform follows these principles:

| Principle | Policy |
|---|---|
| **Impact-Based Gating** | Decisions with high impact require human approval before execution. The earlier a decision is in the SDLC lifecycle, the greater its impact. |
| **Destructive Action Gating** | Deletion of existing files always requires explicit human confirmation. |
| **Parallel Progress** | If a human decision is pending, work continues in parallel on all non-blocked items. The platform never idle-waits when there are unblocked tasks available. |
| **Merge-Time Verification** | AI-assisted human verification of code changes happens at the time of merging a feature branch with the main product branch. |
| **Ad-Hoc Verification** | All other human verification is on an ad-hoc basis, manually initiated outside of the platform's workflow. |

---

## 5. Error Handling & Recovery (SOTA-Informed)

The platform's error handling draws from SOTA best practices including the **MAST (Multi-Agent System Failure Taxonomy)** framework, **isolation boundaries**, and **sequenced recovery**.

### Error Handler Agent

A dedicated **Error Handler Agent** is part of every swarm topology. Its responsibilities:

1. **Detection & Logging** — Captures the full failure context: which agent failed, what it was doing, the input it received, the error produced, and the system state at the time of failure.
2. **Root Cause Analysis** — Performs automated RCA: was this a model error (hallucination, reasoning failure), a tool error (API timeout, permission denied), a coordination error (conflicting writes, deadlock), or a resource error (context overflow, quota exhaustion)?
3. **Routing Decision** — Based on the RCA, the Error Handler decides:
   - **Retry** — If transient (e.g., API timeout), retry with backoff.
   - **Reassign** — If agent-specific (e.g., model confusion), route to a different agent or model.
   - **Workaround** — If a known workaround exists in project memory, apply it.
   - **Escalate to Human** — If the failure is novel, high-impact, or no single agent is qualified to resolve it, compile a clear handover report and escalate.
4. **Isolation** — Failed agents are isolated to prevent cascading failures. The platform uses isolation boundaries grouped by business capability.
5. **Sequenced Recovery** — Recovery follows the dependency graph to avoid overloading the system or causing new failures.

> [!IMPORTANT]
> This approach is **aligned with SOTA best practices**. Key references:
> - The **MAST framework** (Multi-Agent System Failure Taxonomy) categorizes failures into coordination breakdowns, weak specifications, and misaligned roles — enabling systematic detection and routing.
> - **Isolation boundaries** (grouping agents around business capabilities) prevent single-agent failures from destabilizing the entire swarm.
> - **Sequenced recovery** using dependency graphs is a proven pattern for restoring multi-agent systems without cascading failures.
> - The "route or escalate" decision tree (retry → reassign → workaround → escalate) is consistent with hierarchical error handling patterns in production multi-agent deployments.

---

## 6. Budget & Context Management Agent

A dedicated **Budget Agent** continuously monitors and manages token consumption and context window health across the swarm.

### Responsibilities

| Function | Description |
|---|---|
| **Context Window Monitoring** | Tracks each agent's context window utilization in real time. Alerts when agents approach capacity. |
| **Token Quota Tracking** | Monitors overall token usage against the session/daily/monthly quota. Projects burn rate and estimates remaining capacity. |
| **Proactive Handover Planning** | When context window bloat or quota exhaustion is imminent, the Budget Agent advises the relevant agents (determined by swarm topology) to plan and execute a **work thread handover** — seamlessly transferring the task to a fresh agent with a compressed context summary. |
| **Optimization Advisory** | Identifies opportunities for token optimization (see §3.1) and recommends/implements them: converting repetitive tasks to scripts, switching to context bundles, compressing intermediate memory. |
| **Cost Reporting** | Provides transparency into where tokens were spent: by agent role, by journey phase, by task type. Enables continuous improvement. |

### Seamless Handover Protocol

When a handover is triggered (context bloat or quota exhaustion):

1. The active agent receives a "prepare for handover" signal.
2. The active agent compresses its current state into a structured handover document (stored in session memory).
3. A fresh agent is spun up with the handover document as its starting context.
4. The fresh agent validates it has sufficient context to continue, requests additional context slices if needed, and resumes work.
5. The original agent's working memory is archived.

> [!TIP]
> The handover is designed to be **invisible to the work outcome** — the user experiences uninterrupted progress. The logistics of context management and quota management are abstracted away from both the user and the working agents.

---

## 7. Learning & Improvement (SOTA-Informed)

The platform continuously improves its strategies, topology selection, and agent performance based on outcomes and feedback. This draws from SOTA research on continuous self-improvement in multi-agent LLM systems.

### Learning Mechanisms

| Mechanism | Description | SOTA Basis |
|---|---|---|
| **Outcome-Based Feedback Loops** | After each journey completes, the platform evaluates the outcome (success/failure, quality metrics, token cost, time elapsed) and stores the evaluation in organizational memory. Future topology and strategy selection is informed by historical performance data. | Iterative feedback loops where agents refine strategies based on outcomes (NeurIPS 2024 self-improvement research). |
| **Successful Solution Caching** | When an agent solves a task effectively, the solution approach (not just the output) is cached as an in-context example for future similar tasks. This reduces token cost and improves quality for recurring patterns. | Storage and reuse of successful solutions as in-context examples (Substack/research consensus). |
| **Role Specialization Through Experience** | Over time, agents that consistently perform well on specific task types are flagged as specialists. The topology selection engine preferentially assigns them to matching tasks. | MALT (Multi-Agent LLM Training) — training specialized agents for distinct roles based on correct and incorrect trajectories. |
| **Post-Journey Retrospectives** | After significant journeys, a Retrospective Agent automatically generates a brief analysis: what went well, what bottlenecked, what caused rework. These retrospectives are aggregated to identify systemic improvement opportunities. | Meta-learning agents that analyze and optimize their own task-solving methods (classicinformatics.com research). |
| **Topology Performance Tracking** | The platform tracks which topology was selected for each task type and how well it performed. Over time, topology selection heuristics are refined based on empirical data from the user's specific workload patterns. | Adaptive architecture research where systems dynamically adjust based on performance feedback. |
| **Cross-Journey Pattern Mining** | The platform mines organizational memory for patterns across journeys — e.g., "bugs in module X often stem from data model mismatches" — and proactively surfaces these insights to relevant agents. | Society of Minds research — diverse specialized models that improve through multi-agent interaction. |

---

## 8. Deferred & Out-of-Scope Items

| Item | Status | Notes |
|---|---|---|
| **Multi-Tenant / Multi-Team Memory Boundaries** | Deferred | Not a priority; platform is for personal use. |
| **Observability Dashboard** | Separate Session | To be brainstormed in a dedicated session. |
| **Latency vs. Quality Trade-offs** | Resolved | Quality is non-negotiable. All work is thorough. No "fast mode". |

---

## 9. Resolved Design Decisions

### 9.1 Topology Selection — Human-Confirmed by Default

The platform uses **human-confirmed topology selection** as the default:

1. The platform analyzes the task and selects a recommended topology.
2. The platform presents the recommendation to the user with:
   - The selected topology name and visual structure.
   - **Rationale**: Why this topology fits the task (task shape, dependencies, parallelism opportunities).
   - **Trade-offs**: What alternatives were considered and why they were ranked lower.
3. The user **confirms** the recommendation or **overrides** with a different topology.
4. Execution begins only after human confirmation.

> [!NOTE]
> This is a checkpoint, not a blocker. Work on non-topology-dependent preparation (context retrieval, memory loading) can proceed in parallel while the user reviews the topology recommendation.

---

### 9.2 Cross-Journey Agent Continuity — Resident Agents

Certain agents **persist across journeys** within the same project, functioning as "resident" specialists:

| Resident Agent | Persistence Scope | Purpose |
|---|---|---|
| **Resident Architect** | Project lifetime | Maintains deep understanding of the system's architecture, ensures consistency across features, catches drift |
| **Resident Standards Agent** | Project lifetime | Accumulates project conventions and coding standards knowledge, enforces them consistently |
| **Resident Context Agent** | Project lifetime | Maintains a living index of the codebase, reducing context-loading cost for all transient agents |

Transient agents (workers, testers, reviewers) are created per-journey and discarded after completion. Their outputs are persisted in project memory, not in the agents themselves.

---

### 9.3 Deterministic Automation Threshold — 3 Repetitions (Configurable)

When the platform detects that the same task pattern has been performed by agents **3 times**, it automatically proposes converting that task into a deterministic script.

- **Default threshold**: 3 repetitions.
- **Configurable**: The user can adjust this threshold (e.g., set to 2 for aggressive optimization, or 5 for more conservative conversion).
- **Human confirmation**: The platform proposes the script conversion with a preview; the user confirms before the script replaces the agent-driven approach.
- **Rollback**: If a converted script starts failing on edge cases, the platform reverts to agent-driven execution and logs the failure pattern to refine the script.

---

### 9.4 Human Override Decision Logging — Manual Review, No Auto-Learning

When a human overrides an agent's decision at a checkpoint:

1. **Full Context Logging** — The platform logs:
   - The original agent recommendation and its reasoning.
   - The human's override decision.
   - The human's rationale (the agent explicitly asks for rationale if not proactively provided).
   - The journey context at the time of the override.
2. **No Automated Propagation** — Override decisions are **never** automatically used by the platform's self-improvement mechanisms. The platform does not learn from overrides autonomously.
3. **Human-Driven Review** — The growing log of override decisions is available for the human user to periodically review. The user independently decides whether the platform needs improvements based on patterns they observe.

> [!IMPORTANT]
> This is a deliberate design choice. Automated learning from overrides risks amplifying one-off human preferences or context-specific decisions into permanent behavioral changes. Human review ensures only intentional, well-considered improvements are applied.

---

## 10. Memory Retention — Hybrid: Persist Everything, Compress Progressively (Resolved)

Given the user's profile — **small projects** with **prolonged inactivity gaps** — the platform uses a hybrid approach that combines the safety of indefinite persistence with the efficiency of progressive compression.

### Core Principle

> **Nothing is ever deleted. Everything is progressively compressed. Retrieval is always on-demand via RAG.**

### How It Works

```
Journey completes
    │
    ▼
┌─────────────────────────────────────────────┐
│  HOT MEMORY (Session Memory)                │
│  Full-fidelity raw data                     │
│  Duration: 7 days after journey completion  │
│  Access: Directly loaded into agent context │
└──────────────────┬──────────────────────────┘
                   │ after 7 days
                   ▼
┌─────────────────────────────────────────────┐
│  WARM MEMORY (Project Memory)               │
│  Compressed: key decisions, patterns,       │
│  outcomes, ADRs — raw data preserved but    │
│  not loaded by default                      │
│  Duration: Indefinite                       │
│  Access: RAG retrieval (semantic search)    │
└──────────────────┬──────────────────────────┘
                   │ periodic deduplication
                   ▼
┌─────────────────────────────────────────────┐
│  COLD ARCHIVE (Organizational Memory)       │
│  Full raw data preserved permanently        │
│  Deduplicated & indexed                     │
│  Access: On-demand deep retrieval           │
└─────────────────────────────────────────────┘
```

### Key Properties

| Property | Behavior |
|---|---|
| **Nothing is deleted** | Raw data is always preserved in cold archive. Safe for projects revisited after months of inactivity. |
| **Progressive compression** | Memory is summarized as it ages — agents work with concise, high-signal context rather than verbose raw logs. |
| **RAG-powered retrieval** | Warm and cold memory items are retrieved via semantic similarity, so only relevant context surfaces. Agents aren't overloaded. |
| **Small project optimization** | Since projects are small, storage growth is bounded. The platform can afford to keep everything without significant cost. |
| **Long-gap resilience** | When a project is revisited after months, compressed warm memory provides a fast "catch-up" context. Full raw details are available in cold archive if needed. |
| **Deduplication** | Periodic passes merge redundant memory items across journeys to keep the store lean. |
