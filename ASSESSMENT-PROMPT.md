# Pattern Assessment Prompt

> Copy this prompt and give it to your AI agent along with access to your codebase, configuration, and system documentation. In about ten minutes it will tell you which Open Patterns your system already implements, which ones you're missing, and what broke for the people who were missing them.

This is the fastest way to get value from the catalog. You don't have to read 36 patterns. Your agent reads them and reports back against *your* system.

---

## The Prompt

```
I want you to assess our system against the Open Patterns Initiative catalog
(https://github.com/cloudnirvana/open-patterns), a set of production-tested design
patterns for AI agents, cloud, and technology implementation.

First, read these files from the Open Patterns repo:
1. patterns.yaml — the catalog index. Read the full list of patterns AND the list of
   categories from this file; do not assume the categories from memory.
2. AI-GUIDE.md — how the catalog is structured and what "published" vs "in-progress" means.
3. patterns/trust-governance/ladder-of-trust.md — one reference pattern, so you know
   what a "What Broke in Practice" section looks like.

Then scan our system: codebase, agent configurations, tool policies, deployment scripts,
runbooks, and architecture docs. For EVERY pattern in patterns.yaml, classify our system
as one of:

- IMPLEMENTED    — we do this, and you found the mechanism
- PARTIAL        — we do part of it, or we do it inconsistently
- MISSING        — the pattern applies to us and we don't do it
- NOT APPLICABLE — the pattern doesn't apply to our architecture (say why)

For every IMPLEMENTED or PARTIAL classification, state the EVIDENCE and classify it:
- CONFIG    — enforced by configuration, tool policy, or infrastructure (the system
              cannot do otherwise)
- CODE      — enforced by a script, validator, or deterministic check
- PROMPT    — stated only in a prompt, system instruction, or runbook (the agent is
              asked to comply, but nothing prevents it from doing otherwise)
- NONE      — asserted somewhere but you could not find any enforcement

Be strict about this. A rule written in a prompt is a suggestion, not an
implementation. Do not classify a pattern as IMPLEMENTED on PROMPT evidence alone;
classify it as PARTIAL and say so. This distinction is the point.

For every MISSING pattern that applies to us, read that pattern's "What Broke in
Practice" section and summarize, in two sentences, what happened to the contributor
who was missing it. That is the cost of the gap.

Produce a report with:
1. A summary table: pattern | category | status | evidence type | file/line where found
2. The top 5 gaps, ranked by the severity of that pattern's "What Broke," with the
   two-sentence cost for each
3. Every pattern you classified as IMPLEMENTED on PROMPT evidence only, listed
   separately as "speed bumps": rules we have that nothing enforces
4. Anything in our system that looks like a pattern the catalog doesn't have yet
   (a candidate for contribution; see DISCOVERY-PROMPT.md)

Do not soften the findings. Do not credit us for things you could not find evidence
for. If you can't reach a file you need, say which one.
```

---

## What You Get Back

A gap analysis of your system against every pattern in the catalog, with the cost of each gap stated in the words of someone who paid it. Three parts of the report are worth your attention in this order:

**The speed-bump list (section 3).** These are the controls you *think* you have. Every one is a rule that lives in a prompt with nothing behind it. This list is usually the most uncomfortable and the most useful.

**The top gaps (section 2).** Ranked by how badly it went for the people who lacked them. Start with the one whose "What Broke" reads most like your system.

**The contribution candidates (section 4).** If your agent found something that works and isn't in the catalog, that's a pattern the field is missing. See [DISCOVERY-PROMPT.md](DISCOVERY-PROMPT.md) to draft it, and [CONTRIBUTING.md](CONTRIBUTING.md) to submit it.

---

## Notes

- **Works with any capable agent** that can read a public GitHub repo and your local files: coding assistants, agentic frameworks, or a chat model with file access.
- **Point it at the real config, not the docs.** The evidence classification only means something if the agent reads tool policies, allowlists, and scripts, not the architecture diagram that says what they should be.
- **Run it again after you change things.** The report is a snapshot. The catalog grows, and so does your drift.
- **In-progress patterns** are marked as such in `patterns.yaml`. Your agent should still assess against them; the failure modes in those are real even where the fix isn't finished.

*Every pattern in the catalog has a mandatory "What Broke in Practice" section. That's what makes this assessment worth running: the gaps come with receipts.*
