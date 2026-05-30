# Cron as Task Runner, Not Task Definer

> **One-line intent:** Work definition lives in the WMS (Notion), trigger logic lives in cron, execution lives in the agent — crons are generic runners, not hardcoded task prompts.

## Pattern in 60 Seconds

_The entire pattern distilled into something anyone can read in under a minute. No jargon, no code. A CEO, an engineer, and an entrepreneur should all understand this section._

**The problem:** Teams define agent tasks directly in cron configs — hardcoded prompts like "every morning at 8am, draft a briefing email." When task definitions change, someone has to find and edit cron configs. The work definition is buried in infrastructure.

**The insight:** Cron should be a dumb runner. It fires at a schedule, asks "what work is assigned to me?", and executes whatever the WMS (work management system) says. Task definitions live in Notion (or equivalent). Cron is just the trigger. The agent is just the executor.

**The key structure:**

| Layer | Responsibility | Example |
|-------|---------------|---------|
| WMS (Notion) | Defines what work exists, assigned to whom, with what parameters | Task: "Draft Columbus event invite" → assigned to Lou-i, due Monday |
| Cron | Fires on schedule, queries WMS for assigned tasks, passes to agent | `0 8 * * *` → query Notion → spawn agent with task list |
| Agent | Reads task definition from WMS, executes, reports outcome back to WMS | Reads task params, drafts email, marks Done |

**Three trigger patterns:**
- **Scheduled:** Cron fires on cadence, agent checks WMS for work
- **Event-Driven:** Dispatcher spawns agent when new work appears in WMS
- **Human-Initiated:** Human creates task in Notion; agent picks up on next run

**What broke when we got this wrong:** A task prompt was hardcoded in a cron config. The task requirements changed. Nobody updated the cron. The agent kept executing the old task definition for a week before anyone noticed.

---

## Classification

| Property | Value |
|----------|-------|
| **Category** | Operations & Orchestration |
| **Difficulty** | Intermediate |
| **Also Known As** | WMS-Driven Execution, Decoupled Task Orchestration, Scheduler-Runner Separation |

---

## Motivation

On April 10, 2026, while auditing the Cloud Nirvana AIOS cron configuration, a pattern became visible that felt immediately wrong: several cron jobs had hardcoded task prompts embedded in their config. One job said, in effect, "every morning, tell the agent to do X." Another said "every Monday, generate Y." These were reasonable tasks at the time they were written. But task requirements evolve. The "X" had changed. The cron hadn't.

The deeper problem wasn't the stale prompt — it was the architectural coupling. When task definitions live in cron configs, they're invisible to the people who manage work. A project manager updates the task in Notion. The cron is never touched. The agent keeps executing the old definition. The WMS says one thing; the actual execution does another. This divergence is silent and dangerous, especially for recurring operational tasks like weekly digests, event reminders, or campaign sends.

The fix is to invert the relationship between scheduling and work definition. Cron becomes a generic trigger: it fires, queries the WMS for tasks assigned to the current agent/schedule, and passes those tasks to the agent for execution. The task definition — what to do, with what parameters, for which recipients, by when — lives entirely in the WMS. Changing a task means updating Notion, not editing a cron file. Adding a new task means creating a Notion record, not provisioning infrastructure.

This pattern also enables three distinct trigger models from the same architecture. Scheduled triggers are the cron case: predictable cadence, WMS query on each fire. Event-driven triggers skip the cron entirely — when a new task appears in the WMS, a dispatcher notices and spawns an agent immediately. Human-initiated triggers are the most common: a person creates or updates a task record in Notion, and the agent picks it up on the next scheduled run. All three models work with the same agent and the same WMS. Only the trigger mechanism differs. This makes the system composable: the same agent that runs on a schedule can also respond to ad-hoc human requests, without any code changes.

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

```yaml
# TODO: cron config example showing generic runner pattern
```

---

## Consequences

### Benefits
- TODO

### Liabilities
- TODO

### What Broke in Practice
- Hardcoded task prompt in cron config → requirements changed → cron not updated → agent ran stale definition for a week silently
- No single source of truth for "what is this agent supposed to be doing" — split between Notion and cron configs
- Adding new tasks required infrastructure changes (editing cron) rather than WMS record creation

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
| Cloud Nirvana | AIOS cron architecture — Lou-i (operational agent) queries Notion Tasks for assigned work on each scheduled run | Team |

---

## Related Patterns

| Pattern | Relationship |
|---------|-------------|
| Agent Lifecycle Management | Task assignment in WMS should respect agent's current trust tier — lifecycle governs what the agent is authorized to execute |
| Checkpoint-Gated Autonomy for Stateless Agents | WMS task status can serve as the approval signal that triggers the execution-phase session |
| Fit-for-Purpose Model Routing | Task type metadata from WMS informs model tier selection for execution |
| Memory vs. Authority Boundary | Task parameters from WMS are System of Record — agent reads them fresh each run, never from session memory |

---

## Metadata

| Property | Value |
|----------|-------|
| **Contributor** | Sean Erikson, Cloud Nirvana |
| **Production Environment** | Cloud Nirvana AIOS, macOS, OpenClaw, Notion WMS, small team |
| **First Published** | 2026-05-29 |
| **Last Updated** | 2026-05-29 |
| **Cloud Nirvana Event** | TBD |
| **License** | CC BY 4.0 |

---

## Revision History

| Date | Change | Author |
|------|--------|--------|
| 2026-05-29 | Inception stub — origin story, 60 seconds, classification, motivation | Lou / Sean Erikson |
