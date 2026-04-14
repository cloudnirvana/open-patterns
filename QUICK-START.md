# Open Patterns — Quick Start

> **One file. Everything you need.** Copy this into any AI tool (Claude, ChatGPT, Copilot, Cursor, or anything else) and start identifying and drafting patterns.

---

## What Is This?

The Open Patterns Initiative is an open-source catalog of engineering patterns for AI, cloud, and technology implementation. Patterns are practitioner-sourced, tested in production, and documented honestly (including what broke).

**Repo:** [github.com/cloudnirvana/open-patterns](https://github.com/cloudnirvana/open-patterns)

---

## The 7 Pattern Categories

| # | Category | What It Covers |
|---|----------|---------------|
| 1 | **Trust & Governance** | Permissions, approvals, human oversight, progressive autonomy |
| 2 | **Agentic Architecture** | Agent coordination, orchestration, state management |
| 3 | **RAG & Knowledge** | Retrieval, chunking, search, knowledge management |
| 4 | **Production Readiness** | Failures, fallbacks, offline operation, deployment |
| 5 | **Data Quality** | Identity resolution, dedup, enrichment, validation |
| 6 | **Cost & Operations** | Token management, monitoring, scaling, scheduling |
| 7 | **Security & Compliance** | Credentials, PII, trust boundaries, audit |

---

## Pattern Template

Every pattern follows this structure. Copy it and fill it in.

```markdown
# [Pattern Name]

> **One-line intent:** [What does this pattern do, in one sentence?]

## Pattern in 60 Seconds

**The problem:** [One sentence]
**The insight:** [One sentence — the core idea]
**The key structure:** [A simple table or 3-4 bullets]
**What broke when we got this wrong:** [One concrete example]

## Classification

| Property | Value |
|----------|-------|
| **Category** | [from the 7 categories above] |
| **Difficulty** | [Foundational / Intermediate / Advanced] |
| **Also Known As** | [Alternative names, if any] |

## Motivation
[2-4 paragraphs: a concrete scenario illustrating the problem. Make the reader say "I've been there."]

## Applicability
Use this pattern when:
- [condition]

Do NOT use this pattern when:
- [condition]

## Structure
[Mermaid diagram showing the pattern's components and relationships]

## Participants
| Participant | Role | Example |
|------------|------|---------|
| [Component] | [What it does] | [Concrete example] |

## How It Works
[Step-by-step narrative + code/config examples]

## Consequences
### Benefits
- [Specific benefit]

### Liabilities
- [What you give up]

### What Broke in Practice
[MANDATORY. Real failure modes. No hero narratives.]

## Implementation Notes
### Variations
- [Alternative approaches]

### Common Pitfalls
- [Mistakes to avoid]

## Security Implications
### Attack Surface
- [How this pattern affects security]

### Mitigations
- [Controls and defenses]

## Known Uses
| Organization | Context | Scale |
|-------------|---------|-------|
| [Who] | [How they use it] | [Scale] |

## Related Patterns
| Pattern | Relationship |
|---------|-------------|
| [Name] | [How it connects] |

## Metadata
| Property | Value |
|----------|-------|
| **Contributor** | [Name, Title, Company] |
| **First Published** | [Date] |
| **License** | CC BY 4.0 |
```

---

## Example: Ladder of Trust (Condensed)

> **One-line intent:** Incrementally grant an AI system more autonomy by earning trust through demonstrated reliability at each level.

**Four levels:**

| Level | Name | What Happens |
|-------|------|-------------|
| 1 | Supervised (Propose) | AI recommends. Human decides and acts. |
| 2 | Guided (Execute with Guardrails) | AI acts within boundaries. Exceptions escalate. |
| 3 | Autonomous (Operate) | AI owns its domain. Human monitors patterns. |
| 4 | Strategic (Evolve) | AI improves its own domain. Human governs changes. |

**Key rules:**
- Trust is per-action-type, not global (same agent can be Level 1 for finances and Level 3 for data analysis)
- Bright lines: some actions never leave Level 1 regardless of reliability
- Trust is earned slowly and lost quickly

**What broke:** An agent moved 107 emails on day two before earning the trust to do so. Nothing was lost, but it proved that skipping levels creates risk even when the system is technically capable.

**Full pattern:** [patterns/trust-governance/ladder-of-trust.md](https://github.com/cloudnirvana/open-patterns/blob/main/patterns/trust-governance/ladder-of-trust.md)

---

## Discovery Prompts

### If Your AI Tool Has Codebase Access
(Claude Code, Copilot, Cursor, OpenClaw, Windsurf, etc.)

```
Read the Open Patterns catalog at github.com/cloudnirvana/open-patterns
(specifically patterns.yaml and PATTERN-TEMPLATE.md).

Then scan our codebase and identify implementations that match any of the
9 pattern categories. For each pattern you find:

1. Draft a pattern card following the template
2. Include real code/config examples from our system
3. Be honest in "What Broke in Practice"
4. Fill in "Security Implications"
5. Use Mermaid for diagrams

Also identify pattern gaps — categories where we have no implementation.

Output: A list of identified patterns, full drafts for the top 3, and a gap list.
```

### If Your AI Tool Is Browser-Based
(Claude.ai, ChatGPT, Perplexity, etc.)

```
I'm going to describe my system's architecture. I want you to identify engineering
patterns that could be contributed to the Open Patterns Initiative
(github.com/cloudnirvana/open-patterns).

The catalog has 9 categories:
1. Trust & Governance (permissions, approvals, progressive autonomy)
2. Agentic Architecture (agent coordination, orchestration, state)
3. RAG & Knowledge (retrieval, chunking, search)
4. Production Readiness (failures, fallbacks, deployment)
5. Data Quality (identity resolution, dedup, enrichment)
6. Organizational Readiness (team onboarding, governance)
7. Integration (legacy systems, APIs)
8. Cost & Operations (token management, monitoring, scaling)
9. Security & Compliance (credentials, PII, trust boundaries)

Here's my system:
[PASTE YOUR ARCHITECTURE DESCRIPTION, README, OR KEY CONFIG FILES HERE]

For each pattern you identify:
1. Give it a name and one-line intent
2. Draft the "Pattern in 60 Seconds" section (plain language, no jargon)
3. Draft "What Broke in Practice" (or potential failure modes if you can identify them)
4. Draft "Security Implications"
5. Classify it by category and difficulty

Then identify pattern gaps — categories where my system has no clear implementation.
```

### Quick Pattern Brainstorm
(Any tool, no codebase needed)

```
I built [DESCRIBE WHAT YOU BUILT IN 2-3 SENTENCES].

Based on the Open Patterns Initiative categories (Trust & Governance, Agentic Architecture,
RAG & Knowledge, Production Readiness, Data Quality, Organizational Readiness, Integration,
Cost & Operations, Security & Compliance):

1. What patterns am I likely using, even if I haven't named them?
2. What's the most interesting/unique pattern in my system?
3. Draft a "Pattern in 60 Seconds" for that pattern.
```

---

## How to Contribute

1. Use one of the prompts above to identify and draft patterns
2. Review and add your own "What Broke" stories (your AI doesn't know where the bodies are buried)
3. Fork [cloudnirvana/open-patterns](https://github.com/cloudnirvana/open-patterns)
4. Add your pattern to the appropriate category directory under `patterns/`
5. Open a pull request

**The one non-negotiable:** Every pattern must include "What Broke in Practice." No hero narratives.

---

*Open Patterns Initiative by [Cloud Nirvana](https://cloudnirvana.org) · CC BY 4.0*
