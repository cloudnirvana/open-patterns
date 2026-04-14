# Pattern Discovery Prompt

> Copy this prompt and give it to your AI agent along with access to your codebase or system documentation. It will scan for patterns and draft pattern cards for your review.

---

## The Prompt

```
I want you to identify engineering and design patterns in our system that could be
contributed to the Open Patterns Initiative (https://github.com/cloudnirvana/open-patterns).

First, read these files from the Open Patterns repo:
1. patterns.yaml — the catalog index and category definitions
2. PATTERN-TEMPLATE.md — the required structure for pattern cards
3. patterns/trust-governance/ladder-of-trust.md — a reference pattern showing the expected quality and depth

Then scan our system (codebase, architecture docs, configuration, deployment scripts) and identify
implementations that match any of these 7 pattern categories:

1. Trust & Governance — permissions, approvals, human oversight, progressive autonomy
2. Agentic Architecture — agent coordination, orchestration, state management
3. RAG & Knowledge — retrieval, chunking, search, knowledge management
4. Production Readiness — failures, fallbacks, offline operation, deployment
5. Data Quality — identity resolution, dedup, enrichment, validation
6. Cost & Operations — token management, monitoring, scaling, scheduling
7. Security & Compliance — credentials, PII, trust boundaries, audit

For each pattern you identify:
1. Draft a complete pattern card following PATTERN-TEMPLATE.md
2. Start with "Pattern in 60 Seconds" (plain language, no jargon)
3. Include real code/config examples from our system
4. Be honest in "What Broke in Practice" — identify actual or potential failure modes
5. Fill in "Security Implications" — how this pattern affects our attack surface
6. Use Mermaid for diagrams

Also identify patterns we SHOULD be using but aren't. These are gaps, not contributions,
but they're equally valuable for improving our system.

Output:
- A list of identified patterns with one-line descriptions
- Full pattern card drafts for the top 3 most interesting/unique ones
- A list of pattern gaps (categories where we have no implementation)
```

---

## Tips for Best Results

- **Give the agent read access to your repo** or paste in relevant architecture docs, config files, and READMEs
- **Start broad, then go deep.** Let the agent identify candidates first, then ask for full drafts of the most interesting ones.
- **Review critically.** The agent may identify "patterns" that are just standard configurations. A real pattern solves a recurring problem in a non-obvious way.
- **Check the "What Broke" section.** If the agent couldn't find real failure modes, add your own. You know where the bodies are buried.
- **Don't skip Security Implications.** If the agent missed an attack vector, add it. Security gaps in published patterns erode trust in the catalog.

## After Discovery

1. Review the drafted pattern cards
2. Add your own "What Broke" stories and security insights
3. Fork `cloudnirvana/open-patterns`
4. Add your patterns to the appropriate category directory
5. Open a pull request

Welcome to the Open Patterns Initiative.
