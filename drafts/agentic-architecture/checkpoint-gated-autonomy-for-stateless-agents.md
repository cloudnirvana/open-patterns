# Checkpoint-Gated Autonomy for Stateless Agents

> **One-line intent:** Agents write a checkpoint file and end their session; when the human approves, a new session spawns, reads the checkpoint, and executes — eliminating timeouts, zombies, and long-running approval waits.

## Pattern in 60 Seconds

_The entire pattern distilled into something anyone can read in under a minute. No jargon, no code. A CEO, an engineer, and an entrepreneur should all understand this section._

**The problem:** Agent does work, presents it for human approval, and then... waits. If approval takes 2 hours, the session times out. If the session ends, the work is lost. Long-running sessions holding state for async human review are fragile, expensive, and architecturally wrong.

**The insight:** Agent sessions are stateless by nature — they end. Design for it. Agent does work → writes checkpoint file → presents for approval → **session ends**. When human approves, a **new session** spawns, reads the checkpoint, and executes. The checkpoint is the handoff mechanism.

**The key structure:**
1. **Work Phase:** Agent completes the pre-approval work, writes checkpoint file with all state needed to execute
2. **Handoff:** Agent presents checkpoint summary to human, ends session
3. **Approval:** Human reviews async — no session holding open, no timeout risk
4. **Execution Phase:** Approval triggers new session → reads checkpoint → executes without re-doing work

**What broke when we got this wrong:** Agent drafted an email and held the session open waiting for approval. Human came back 3 hours later. Session had timed out. Draft was gone. Work had to be redone.

---

## Classification

| Property | Value |
|----------|-------|
| **Category** | Agentic Architecture |
| **Difficulty** | Intermediate |
| **Also Known As** | Stateless Approval Loop, Async Checkpoint Handoff, Session-Safe Autonomy |

---

## Motivation

In May 2026, a recurring problem became visible in the Cloud Nirvana AIOS: agent sessions that required human approval were architecturally mismatched with how humans actually work. The agent would complete a complex task — drafting a partner email, generating a campaign plan, compiling an attendee report — present it for review, and then idle, holding session state, waiting for a response. Human review doesn't happen in seconds. It happens when the human gets to it: after a meeting, after lunch, sometimes hours later.

This created a reliability failure mode. Sessions timed out. Drafts disappeared. Work had to be redone. The agent's state — everything it had computed and held in context — evaporated at session boundary. The solution teams reached for intuitively was to make sessions longer, increase timeout limits, or add keep-alive mechanisms. These are all wrong. They're trying to make a fundamentally stateless system pretend to be stateful.

The right answer is to embrace statelessness and design the approval loop to work across session boundaries. The checkpoint file is the key mechanism: before ending its session, the agent serializes everything needed for execution into a durable artifact on disk. The checkpoint contains the full execution plan — email content, recipient list, timing, verification results, any computed state. The agent then presents a human-readable summary and explicitly ends. When the human approves (via a message, a button, a command), a new session spawns. The new session reads the checkpoint, verifies it's still valid, and executes. No state held in memory across the gap. No timeout risk. No zombie sessions.

This pattern solves more than reliability. It also creates a clean audit trail. The checkpoint file is a durable record of what the agent intended to do, when it intended to do it, and what it actually executed. For high-stakes operations — bulk email sends, database writes, calendar modifications — this audit trail has value independent of the reliability benefit. You can inspect a checkpoint before approving. You can compare what was planned to what was executed. The checkpoint becomes the contract between planning and execution.

---

## Applicability

Use this pattern when:
- TODO

Do NOT use this pattern when:
- TODO

---

## Structure

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

```json
// TODO: checkpoint file schema example
```

---

## Consequences

### Benefits
- TODO

### Liabilities
- TODO

### What Broke in Practice
- Agent held session open waiting for async approval → session timed out → draft lost → work redone
- Keep-alive workarounds introduced complexity without solving the root architecture mismatch
- No checkpoint = no audit trail; execution intent not separable from execution outcome

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
- Checkpoint files may contain sensitive content (email drafts, contact data, campaign details) — must be access-controlled
- TODO additional

### Failure Modes
- TODO

### Mitigations
- TODO

---

## Known Uses

| Organization | Context | Scale |
|-------------|---------|-------|
| Cloud Nirvana | AIOS email drafting and campaign approval flow — checkpoint-gated send pattern (Mode 2 enforcement) | Team |

---

## Related Patterns

| Pattern | Relationship |
|---------|-------------|
| Agent Lifecycle Management | Checkpoint-gated approval is the mechanical implementation of Mode 2 (draft + approve) in the trust ladder |
| Memory vs. Authority Boundary | Checkpoint should capture verified SoR data, not memory — what was verified at checkpoint creation time |
| Cron as Task Runner, Not Task Definer | Cron can trigger the execution phase when approval signal arrives |

---

## Metadata

| Property | Value |
|----------|-------|
| **Contributor** | Sean Erikson, Cloud Nirvana |
| **Production Environment** | Cloud Nirvana AIOS, macOS, OpenClaw, small team |
| **First Published** | 2026-05-29 |
| **Last Updated** | 2026-05-29 |
| **Cloud Nirvana Event** | TBD |
| **License** | CC BY 4.0 |

---

## Revision History

| Date | Change | Author |
|------|--------|--------|
| 2026-05-29 | Inception stub — origin story, 60 seconds, classification, motivation | Lou / Sean Erikson |
