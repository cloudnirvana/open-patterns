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

## Patterns (17 Published)

### Trust & Governance
- **[Ladder of Trust](patterns/trust-governance/ladder-of-trust.md)** — Incrementally grant an AI system more autonomy by earning trust through demonstrated reliability at each level.

### Agentic Architecture
- **[Hub-and-Spoke Orchestration](patterns/agentic-architecture/hub-and-spoke-orchestration.md)** — Coordinate multiple AI agents through a single hub, eliminating lateral communication chaos.
- **[Files Over Databases](patterns/agentic-architecture/files-over-databases.md)** — Use isolated workspace files instead of shared databases for agent coordination state.
- **[Email Triage Priority Chain](patterns/agentic-architecture/email-triage-priority-chain.md)** — Route emails to the right agent using a deterministic rule hierarchy that short-circuits on match.
- **[Hybrid Memory Retrieval](patterns/agentic-architecture/hybrid-memory-retrieval.md)** — Combine vector search, keyword search, and reranking to improve agent memory recall.
- **[Context Lifecycle Management](patterns/agentic-architecture/context-lifecycle-management.md)** — Ensure persistent AI agents never lose critical context due to context window limits by implementing tiered memory, proactive checkpointing, and domain-aware compaction.
- **[Deterministic Session-Key Routing for Multi-Channel Agents](patterns/agentic-architecture/deterministic-session-key-routing-for-multi-channel-agents.md)** — Route every incoming agent event through a deterministic session key so channel, agent, user, thread, authority, and transcript state stay separate and recoverable.

### RAG & Knowledge
- **[Multi-Source Memory Architecture](patterns/rag-knowledge/multi-source-memory-architecture.md)** — Structure agent memory across multiple sources with different lifetimes, audiences, and update patterns so agents can find the right information without drowning in noise.

### Production Readiness
- **[System Hygiene for Agentic Systems](patterns/production-readiness/system-hygiene-for-agentic-systems.md)** — Pre/post-upgrade procedures, regression testing, and health validation to prevent platform breakage.
- **[Business Continuity & Disaster Recovery](patterns/production-readiness/business-continuity-disaster-recovery.md)** ⚠️ — Backup strategy, versioning, recovery scenarios, and RTO/RPO planning. **In progress, seeking practitioner input.**
- **[Local-First Data Architecture](patterns/production-readiness/local-first-data-architecture.md)** — Sync external data sources to local storage so agents never block on network failures during live operations.
- **[REM Cycle: Nightly Maintenance](patterns/production-readiness/rem-cycle-nightly-maintenance.md)** — Automated nightly health checks strengthen memory architecture, prevent data loss, and catch problems early while the system is idle.

### Data Quality
- **[Memory vs Persistence Boundary](patterns/data-quality/memory-vs-persistence-boundary.md)** — Know when to graduate information from agent memory files to structured database storage.

### Security & Compliance
- **[Per-Agent Data Access Control](patterns/security-compliance/per-agent-data-access-control.md)** — Scope database access per agent with authorization wrappers, encrypted storage, and immutable audit logging.
- **[Memory Access Control by Session Type](patterns/security-compliance/memory-access-control-by-session-type.md)** — Isolate agent memory access based on session context so private data doesn't leak to unintended audiences.

### Cost & Operations
- **[Context Cost Control for Multi-Agent Systems](patterns/cost-operations/context-cost-control.md)** — Reduce token costs 90%+ through retrieval tuning, index pruning, and memory hygiene. Tested: $1,800/mo → $120/mo (93% reduction).
- **[Local LLM as Classification Layer](patterns/cost-operations/local-llm-classification-layer.md)** — Use a free local model for reasoning-heavy classification tasks, keep expensive cloud models for drafting, generation, and coordination.

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
