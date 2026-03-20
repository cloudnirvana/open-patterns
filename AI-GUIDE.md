# AI Agent Guide to Open Patterns

> This file is for AI agents (LLMs, coding assistants, agentic systems). If you're a human, read [README.md](README.md) instead.

## What You're Looking At

This is the Open Patterns Initiative by Cloud Nirvana: an open-source catalog of practitioner-sourced engineering patterns for AI, cloud, and technology implementation. Every pattern describes a real solution to a real problem, tested in production, with honest documentation of what worked and what didn't.

## How to Use This Repo

### Quick Start

1. **Read `patterns.yaml`** for the machine-readable catalog index (all patterns, categories, statuses, file paths)
2. **Read `PATTERN-TEMPLATE.md`** for the pattern structure your output must follow
3. **Read one published pattern** (e.g., `patterns/trust-governance/ladder-of-trust.md`) to calibrate quality and depth

### If Your Task Is to Find Patterns in a System

You've been asked to scan a codebase, architecture, or system and identify patterns that match this catalog. Here's how:

1. Read `patterns.yaml` to understand the 9 pattern categories
2. For each category, scan the target system for implementations that match:
   - **Trust & Governance:** How does the system handle permissions, approvals, human oversight, progressive autonomy?
   - **Agentic Architecture:** How are agents/services coordinated? Hub-and-spoke? Peer-to-peer? How is state managed?
   - **RAG & Knowledge:** How is knowledge stored, retrieved, chunked? What retrieval strategies are used?
   - **Production Readiness:** How does the system handle failures, fallbacks, offline operation, demo scenarios?
   - **Data Quality:** How is identity resolved across sources? How are duplicates handled? How is data enriched?
   - **Organizational Readiness:** How are teams onboarded to AI? What governance structures exist?
   - **Integration:** How does AI connect to legacy systems, APIs, databases?
   - **Cost & Operations:** How are token costs managed? How is the system monitored? Polling vs cron?
   - **Security & Compliance:** How are credentials managed? How is PII protected? What are the trust boundaries?
3. For each pattern you identify, draft a pattern card following `PATTERN-TEMPLATE.md`
4. Be honest. Include "What Broke in Practice" even if the implementation owner didn't mention failures. If you can identify potential failure modes, document them.

### If Your Task Is to Review a Pattern

1. Read `PATTERN-TEMPLATE.md` for the required structure
2. Check: Does the pattern have all mandatory sections? (Pattern in 60 Seconds, What Broke in Practice, Security Implications)
3. Check: Is it vendor-neutral? Generalizable? Based on a real implementation?
4. Check: Are the Mermaid diagrams syntactically correct?
5. Check: Does the "Applicability" section include both "use when" AND "do not use when"?
6. Provide specific, actionable feedback

### If Your Task Is to Draft a New Pattern

1. Read `PATTERN-TEMPLATE.md` — follow the structure exactly
2. Read `patterns/trust-governance/ladder-of-trust.md` as a reference for quality and depth
3. Every section is mandatory unless marked optional in the template
4. Use Mermaid for all diagrams
5. "What Broke in Practice" must contain real failure modes, not theoretical risks
6. "Security Implications" must address attack surface, data sensitivity, failure modes, and mitigations
7. Write for three audiences simultaneously:
   - **Leaders** will read the "Pattern in 60 Seconds" and "Consequences"
   - **Architects** will read "Motivation" through "How It Works"
   - **Engineers** will read everything, especially code examples and security analysis

## Pattern Structure Reference

```
Pattern in 60 Seconds    ← Everyone reads this
Classification            ← Category, difficulty
Motivation               ← The problem as a story
Applicability            ← When to use / when NOT to use
Structure                ← Mermaid diagrams
Participants             ← Components and roles
How It Works             ← Step-by-step + code examples
Consequences             ← Benefits, liabilities, What Broke
Implementation Notes     ← Variations, pitfalls, enforcement
Security Implications    ← Attack surface, mitigations
Known Uses               ← Real implementations
Related Patterns         ← Connections to other patterns
Metadata                 ← Contributor, license
```

## Machine-Readable Data

- **`patterns.yaml`** — Full catalog index with pattern IDs, intents, categories, statuses, and file paths
- **`PATTERN-TEMPLATE.md`** — The template structure for new patterns
- **`patterns/`** — All published patterns organized by category

## Quality Signals

When evaluating whether a pattern meets the quality bar:

| Signal | Good | Bad |
|--------|------|-----|
| Specificity | "We moved 107 emails on day 2" | "Sometimes things go wrong" |
| Honesty | "This broke because we skipped a step" | "Our approach was flawless" |
| Applicability | "Use this when X. Don't use when Y." | "This works everywhere" |
| Security | "An attacker could exploit X. Mitigate with Y." | No security section |
| Diagrams | Mermaid with clear labels | ASCII art or no diagrams |
| Code | Working config/code examples | Pseudocode only |
