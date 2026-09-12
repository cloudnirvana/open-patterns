# Open Patterns Initiative
### The Practitioner Catalog for Agentic AI

**Practitioner-sourced patterns for AI, cloud, and technology implementation.**

By [Cloud Nirvana](https://cloudnirvana.org) · CC BY 4.0

---

## What This Is

A living catalog of engineering and design patterns contributed by practitioners who build real systems. Not theory. Not vendor guides. Real implementations, real decisions, real tradeoffs.

Every pattern follows a consistent structure inspired by the Gang of Four, adapted for modern AI and cloud engineering with a practitioner voice.

## How to Read a Pattern

Each pattern has three layers. Read as deep as your role requires:

| Layer | Audience | What you'll find |
|-------|----------|-----------------|
| **Pattern in 60 Seconds** | Everyone | The concept in plain language. No jargon, no code. |
| **Motivation + Structure + Consequences** | Leaders, architects, PMs | The "why," the framework, the tradeoffs. |
| **Full Implementation** | Engineers, implementers | Config examples, diagrams, code, security analysis. |

## Patterns (31 Published, 3 In Progress)

### Trust & Governance

- **[Ladder of Trust](patterns/trust-governance/ladder-of-trust.md)** — Incrementally grant an AI system more autonomy by earning trust through demonstrated reliability at each level.
- **[Checkpoint-Gated Autonomy](patterns/trust-governance/checkpoint-gated-autonomy.md)** — Decouple AI agent work from human approval using durable state, so agents don't need to stay alive while humans decide.
- **[Training Email Passthrough](patterns/trust-governance/training-email-passthrough.md)** — Self-addressed emails with 'Training' in the subject bypass production routing, enabling safe agent training without polluting operational queues or triggering real workflows.
- **[Agentic Identity & Lifecycle](patterns/trust-governance/agentic-identity-lifecycle.md)** — Treat agents as a first-class identity type with their own lifecycle, per-capability trust progression, continuous observability, mandatory human ownership, and instant kill switch.
- **[Memory vs. Authority Boundary](patterns/trust-governance/memory-vs-authority-boundary.md)** — Agents must treat memory as a hypothesis and datastores as truth — every factual claim in an external communication must be verified against the System of Record before use.
- **[Operator Discipline](patterns/trust-governance/operator-discipline.md)** — The human operator must refuse the fast path, not doing operational work directly through a high-privilege agent even when it's faster, because the fast path bypasses the safeguards. The behavioral constraint you run while you build the architectural one.
- **[Maker / Checker for Agents](patterns/trust-governance/maker-checker-for-agents.md)** — One agent produces the work and a second agent or a human reviews it before it ships, and the corrections are logged so quality is measurable and trust is earned.

### Agentic Architecture

- **[Hub-and-Spoke Agent Orchestration](patterns/agentic-architecture/hub-and-spoke-orchestration.md)** — Coordinate multiple AI agents through a single hub, eliminating lateral communication chaos.
- **[Strategic Session Bridge](patterns/agentic-architecture/strategic-session-bridge.md)** — Synchronize operational context (cheap model) with strategic reasoning (expensive model) through automated bidirectional data flow, eliminating the human clipboard.
- **[Thread Continuity Routing](patterns/agentic-architecture/thread-continuity-routing.md)** — Route email replies to match a thread's existing agent assignment, not just keyword matching, preserving conversation context across multi-message exchanges.
- **[Files Over Databases for Agent State](patterns/agentic-architecture/files-over-databases.md)** — Use isolated workspace files instead of shared databases for agent coordination state.
- **[Email Triage with Priority Chain](patterns/agentic-architecture/email-triage-priority-chain.md)** — Route emails to the right agent using a deterministic rule hierarchy that short-circuits on match.
- **[Hybrid Memory Retrieval](patterns/agentic-architecture/hybrid-memory-retrieval.md)** — Combine vector search, keyword search, and reranking to improve agent memory recall.
- **[Context Lifecycle Management](patterns/agentic-architecture/context-lifecycle-management.md)** — Ensure persistent AI agents never lose critical context due to context window limits by implementing tiered memory, proactive checkpointing, and domain-aware compaction.

### Operations & Orchestration

- **[Setlist-Driven Groove](patterns/operations-orchestration/setlist-driven-groove.md)** — Separate the clock from the brain — one cron fires on schedule, one editable runbook defines what the agent does, and configuration lives in a datastore.
- **[Plan of the Day](patterns/operations-orchestration/plan-of-the-day.md)** — Synthesize multiple business event calendars, runbook playbooks, and in-flight work into a single daily executable plan with RACI ownership for every team member.
- **[RACI-Scoped Notifications](patterns/operations-orchestration/raci-scoped-notifications.md)** — Control operational notification volume in multi-agent systems by routing messages based on each person's RACI role per task.
- **[Escalation Chain with SLA](patterns/operations-orchestration/escalation-chain-with-sla.md)** — Ensure AI agents surface blockers within a time-bound window instead of silently stalling, retrying, or hallucinating workarounds.
- **[EOD Reconciliation](patterns/operations-orchestration/eod-reconciliation.md)** — Bridge the gap between work that happened and task status by cross-referencing open tasks against evidence sources at end of day.
- **[Cron as Task Runner, Not Task Definer](patterns/operations-orchestration/cron-as-task-runner-not-task-definer.md)** — Work definition lives in the WMS (Notion), trigger logic lives in cron, execution lives in the agent — crons are generic runners, not hardcoded task prompts.

### RAG & Knowledge

- **[Multi-Source Memory Architecture](patterns/rag-knowledge/multi-source-memory-architecture.md)** — Structure agent memory across multiple sources with different lifetimes, audiences, and update patterns so agents can find the right information without drowning in noise.

### Production Readiness

- **[System Hygiene for Agentic Systems](patterns/production-readiness/system-hygiene-for-agentic-systems.md)** — Establish systematic pre/post-upgrade procedures, regression testing, and health validation to prevent production breakage when the underlying platform changes.
- **[Quality Gate Checkpoint](patterns/production-readiness/quality-gate-checkpoint.md)** — All agents verify drafts against a shared quality checklist before notifying a human, preventing low-quality drafts from reaching the review queue.
- **[Threshold-Gated Observability for AI Operating Systems](patterns/production-readiness/threshold-gated-observability.md)** :warning: — Monitor operational signals across an autonomous AI system, fire alerts only on meaningful state transitions, and surface health to every dashboard through one shared contract. **In progress, seeking practitioner input.**
- **[Human Recovery Path](patterns/production-readiness/human-recovery-path.md)** :warning: — An autonomous system needs a recovery path a capable human can execute when the operator is not at the keyboard, and it has to be real infrastructure, not the luck of the right person being reachable. **In progress, seeking practitioner input.**
- **[Business Continuity & Disaster Recovery for Agent Systems](patterns/production-readiness/business-continuity-disaster-recovery.md)** :warning: — Ensure agent-dependent operations can resume within acceptable timeframes when platform failures, data corruption, or human error cause production outages. **In progress, seeking practitioner input.**
- **[Local-First Data Architecture](patterns/production-readiness/local-first-data-architecture.md)** — Sync external data sources to local storage so agents never block on network failures during live operations.
- **[REM Cycle: Nightly Maintenance for Agent Systems](patterns/production-readiness/rem-cycle-nightly-maintenance.md)** — Automated nightly health checks strengthen memory architecture, prevent data loss, and catch problems early while the system is idle.

### Data Quality

- **[Memory vs Persistence Boundary](patterns/data-quality/memory-vs-persistence-boundary.md)** — Know when to graduate information from agent memory files to structured database storage.

### Security & Compliance

- **[Per-Agent Data Access Control](patterns/security-compliance/per-agent-data-access-control.md)** — Scope database access per agent with authorization wrappers, encrypted storage, and immutable audit logging.
- **[Memory Access Control by Session Type](patterns/security-compliance/memory-access-control-by-session-type.md)** — Isolate agent memory access based on session context so private data doesn't leak to unintended audiences.

### Cost & Operations

- **[Cron-Driven Agent Execution](patterns/cost-operations/cron-driven-agent-execution.md)** — Agents execute on schedule (cron), not on-demand, to batch work, prevent race conditions, and enable autonomous operations without human triggers.
- **[Context Cost Control for Multi-Agent Systems](patterns/cost-operations/context-cost-control.md)** — Reduce token costs 90%+ through retrieval tuning, index pruning, and memory hygiene.
- **[Local LLM as Classification Layer](patterns/cost-operations/local-llm-classification-layer.md)** — Use a free local model for reasoning-heavy classification tasks, keep expensive cloud models for drafting, generation, and coordination.

## Blueprints (2 Published)

A pattern solves one problem. A **blueprint** shows how to build a whole capability by composing patterns together. You look at a blueprint and think "that's the thing I want to build."

- **[Operational Groove](blueprints/operational-groove.md)** — Autonomous project management for hybrid human-AI teams. Composes: Setlist-Driven Groove, Plan of the Day, RACI-Scoped Notifications, Escalation Chain with SLA, EOD Reconciliation.
- **[Strategic Program Management](blueprints/strategic-program-management.md)** — Scalable multi-agent program execution with built-in governance. Composes: Operational Groove + Ladder of Trust + Per-Agent Data Access Control + Agentic Identity & Lifecycle + Hub-and-Spoke Orchestration.

## Get Started

**Using any AI tool?** Copy [QUICK-START.md](QUICK-START.md) into your AI tool (Claude, ChatGPT, Copilot, Cursor, whatever). It has everything: the template, an example, and prompts for every tool type.

**Have an AI agent with repo access?** Point it here. It should read these files:
- **[`patterns.yaml`](patterns.yaml)** — Machine-readable catalog index
- **[`AI-GUIDE.md`](AI-GUIDE.md)** — Instructions for AI agents
- **[`DISCOVERY-PROMPT.md`](DISCOVERY-PROMPT.md)** — Prompt for autonomous pattern discovery

Point your agent at this repo. It will know what to do.

## Contributing

We welcome pattern contributions from practitioners (and their AI agents). See [CONTRIBUTING.md](CONTRIBUTING.md) for the template, review process, and quality bar.

**The one non-negotiable:** Every pattern must include a "What Broke" section. No hero narratives. If nothing broke, you haven't tested it in production.

## Quality Bar

A pattern must:
- Describe a **real implementation** (not theoretical)
- Include **What Broke in Practice** (mandatory)
- Be **vendor-neutral** (reference tools, don't sell them)
- Be **generalizable** (applicable beyond one specific situation)
- Be **honest about tradeoffs** (when NOT to use this pattern)

## Diagrams

All diagrams use [Mermaid](https://mermaid.js.org/) (renders natively on GitHub). If viewing on mobile, refer to the summary tables below each diagram.

## License

[Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE)

Anyone can share, adapt, and build on these patterns. Attribution to Cloud Nirvana and the original contributor required.

## About Cloud Nirvana

[Cloud Nirvana](https://cloudnirvana.org) is a practitioner-first technology community across Ohio's I-71 corridor (Columbus, Cleveland, Cincinnati). We run quarterly events, the [Silicon Heartland Sessions](https://cloudnirvana.org/podcast) podcast, and now this open-source patterns catalog.

People before platforms. Discipline over demos. Shared learning over self-promotion.

## Newsletter

Weekly field notes from building the AIOS that powers Cloud Nirvana:
**[cloudnirvana.substack.com](https://cloudnirvana.substack.com)**

Written for one person. You can subscribe.
