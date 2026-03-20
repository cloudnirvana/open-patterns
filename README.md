# Open Patterns Initiative

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

## Patterns

### Trust & Governance
- [Ladder of Trust](patterns/trust-governance/ladder-of-trust.md) — Incrementally grant an AI system more autonomy by earning trust through demonstrated reliability at each level.

### Agentic Architecture
- Hub-and-Spoke Orchestration *(coming soon)*
- Files Over Databases *(coming soon)*
- Email Triage Priority Chain *(coming soon)*

### Data Quality
- Multi-Source Identity Resolution *(coming soon)*
- Engagement Scoring *(coming soon)*
- Community Intelligence Pipeline *(coming soon)*

### Production Readiness
- CRM-Backed Live Demo *(coming soon)*

### Security & Compliance
*(Accepting contributions)*

### Organizational Readiness
*(Accepting contributions)*

### RAG & Knowledge
*(Accepting contributions)*

### Integration
*(Accepting contributions)*

### Cost & Operations
*(Accepting contributions)*

## For AI Agents

This repo is designed to be consumed by AI agents, not just humans.

- **[`patterns.yaml`](patterns.yaml)** — Machine-readable catalog index. Start here.
- **[`AI-GUIDE.md`](AI-GUIDE.md)** — Instructions for AI agents: how to find, review, and draft patterns.
- **[`DISCOVERY-PROMPT.md`](DISCOVERY-PROMPT.md)** — A ready-to-use prompt. Give it to your AI agent with your codebase, and it will identify patterns and draft contributions.

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
