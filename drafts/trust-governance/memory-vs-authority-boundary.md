# Memory vs. Authority Boundary

> **One-line intent:** Agents must treat memory as a hypothesis and datastores as truth — every factual claim in an external communication must be verified against the System of Record before use.

## Pattern in 60 Seconds

_The entire pattern distilled into something anyone can read in under a minute. No jargon, no code. A CEO, an engineer, and an entrepreneur should all understand this section._

**The problem:** An AI agent confidently uses a "fact" it learned in a prior session — a person's title, a company name, a date — in an outbound email. The fact is wrong. It changed. Nobody caught it.

**The insight:** Memory is a cache with no expiry and no integrity check. Datastores are the source of truth. Any factual claim that leaves the system must be sourced from the datastore, not from memory.

**The key structure:**
- **Memory (cache):** What the agent "knows" — fast, stale, unverified
- **System of Record (authority):** CRM, calendar, Notion, filesystem — slow, current, verified
- **Boundary rule:** Every factual claim in an external communication → verify against SoR before use
- **Cache-miss is safe:** If you don't know, query. Guessing is never acceptable.

**What broke when we got this wrong:** Agent called a CISO a "founder" in an outbound email because a prior session had stored that context. The contact's role had changed. The CRM had the correct title. The agent never queried it.

---

## Classification

| Property | Value |
|----------|-------|
| **Category** | Trust & Governance |
| **Difficulty** | Foundational |
| **Also Known As** | Cache Invalidation for Agents, SoR-First Verification, Hypothesis-Before-Claim |

---

## Motivation

On April 9, 2026, a Cloud Nirvana AI agent drafted an outbound email referencing a contact as a "founder." The agent had encountered this label in an earlier session — perhaps from a LinkedIn summary, a prior email, or a note someone had written. By the time the draft reached review, the contact had been a CISO at a different company for over a year. The CRM had the correct title. The agent never asked.

The failure wasn't hallucination in the traditional sense. The agent didn't invent the title from thin air — it retrieved something it had genuinely learned. But learned knowledge has no expiry date in most agent architectures. Memory is written once and read indefinitely. The world moves on. The cache doesn't.

This pattern captures the architectural discipline required to prevent this class of failure: agents operating in high-stakes communication contexts must treat every piece of recalled "knowledge" as a hypothesis pending verification, not a fact ready for use. The System of Record — CRM, calendar, task database, filesystem — is the arbiter. Memory is context. Authority is data.

The distinction matters most when the claim is going somewhere it can't be taken back: an email, a slide, a report, a published document. Once a wrong title reaches a CISO's inbox, the trust damage is done. Verification is cheap. Recovery is not.

---

## Applicability

Use this pattern when:
- TODO

Do NOT use this pattern when:
- TODO

---

## Structure

_Visual representation of the pattern using Mermaid diagrams._

```mermaid
graph TD
    TODO
```

_TODO: diagram caption_

---

## Participants

| Participant | Role | Example |
|------------|------|---------|
| TODO | TODO | TODO |

---

## How It Works

TODO

### Code / Configuration Example

```python
# TODO
```

---

## Consequences

### Benefits
- TODO

### Liabilities
- TODO

### What Broke in Practice
- Agent called a CISO a "founder" in outbound email — title had changed, CRM had correct data, agent never queried
- Trust damage from wrong title in external email; unrecoverable once sent
- Memory entries have no TTL — stale data can persist indefinitely without structural enforcement

---

## Implementation Notes

TODO

### Variations
- TODO

### Common Pitfalls
- TODO

---

## Security Implications

TODO

### Attack Surface
- TODO

### Data Sensitivity
- TODO

### Failure Modes
- TODO

### Mitigations
- TODO

---

## Known Uses

| Organization | Context | Scale |
|-------------|---------|-------|
| Cloud Nirvana | AIOS agent email drafting — CRM verification mandatory before any external communication | Team |

---

## Related Patterns

| Pattern | Relationship |
|---------|-------------|
| Agent Lifecycle Management | Trust ladder defines verification requirements per autonomy tier |
| Fit-for-Purpose Model Routing | Verification queries don't need frontier models; local/mid-tier sufficient |

---

## Metadata

| Property | Value |
|----------|-------|
| **Contributor** | Sean Erikson, Cloud Nirvana |
| **Production Environment** | Cloud Nirvana AIOS, macOS, small team |
| **First Published** | 2026-05-29 |
| **Last Updated** | 2026-05-29 |
| **Cloud Nirvana Event** | TBD |
| **License** | CC BY 4.0 |

---

## Revision History

| Date | Change | Author |
|------|--------|--------|
| 2026-05-29 | Inception stub — origin story, 60 seconds, classification, motivation | Lou / Sean Erikson |
