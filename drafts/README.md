# Open Patterns — Draft Pipeline

Patterns live here from inception through publication. Every pattern starts as a draft, not a backlog note.

---

## Lifecycle Stages

| Status | Meaning | Location |
|--------|---------|---------|
| `inception` | Idea captured, origin story documented, template started | `drafts/` |
| `draft` | Template substantially filled, Motivation + What Broke written | `drafts/` |
| `review` | Complete, ready for peer/community review | `drafts/` |
| `validated` | Production-tested, What Broke is real, reviewed | `drafts/` |
| `published` | Merged to `patterns/`, in patterns.yaml | `patterns/` |

## The Rule

**A pattern cannot be published without:**
1. A real "What Broke" story — not hypothetical
2. A production implementation — not theoretical
3. A Mermaid diagram
4. Security Implications filled out

## Why This Folder Exists

Backlog notes lose context. Six months after the insight, the "why" is gone and you're left with a label. Full draft files — even incomplete ones — preserve:
- The origin story (when and why the pattern was discovered)
- The "what broke" narrative (the real failure that motivated it)
- The implementation details (while they're fresh)
- The security and liability thinking (easy to skip later)

Write the draft the day you discover the pattern. Polish it later.

---

## Current Draft Status

| Pattern | Category | Status | Origin |
|---------|----------|--------|--------|
| [Thread Continuity Routing](agentic-architecture/thread-continuity-routing.md) | Agentic Architecture | `validated` | March 2026 |
| [Cron-Driven Agent Execution](cost-operations/cron-driven-agent-execution.md) | Cost & Operations | `validated` | Feb-Mar 2026 |
| [Training Email Passthrough](trust-governance/training-email-passthrough.md) | Trust & Governance | `validated` | Feb 2026 |
| [Quality Gate Checkpoint](production-readiness/quality-gate-checkpoint.md) | Production Readiness | `validated` | March 2026 |
| [System Hygiene for Agentic Systems](production-readiness/system-hygiene-for-agentic-systems.md) | Production Readiness | `validated` | April 2026 |
| [Metadata-Separated RAG Architecture](rag-knowledge/metadata-separated-rag-architecture.md) | RAG & Knowledge | `draft` | May 29, 2026 |
| [Temporal Relevance Weighting](rag-knowledge/temporal-relevance-weighting.md) | RAG & Knowledge | `draft` | May 29, 2026 |
| [Memory vs Authority Boundary](trust-governance/memory-vs-authority-boundary.md) | Trust & Governance | `inception` | April 2026 |
| [Agent Lifecycle Management](agentic-architecture/agent-lifecycle-management.md) | Agentic Architecture | `inception` | April 2026 |
| [Fit-for-Purpose Model Routing](cost-operations/fit-for-purpose-model-routing.md) | Cost & Operations | `inception` | May 2026 |
