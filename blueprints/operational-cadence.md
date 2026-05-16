# Blueprint: Operational Cadence

> **One-line intent:** Autonomous project management for hybrid human-AI teams through daily task synthesis, RACI ownership, escalation chains, and self-correcting feedback loops.

*"Get shit done." "Alone we go fast, together we go far."* — Ben Blanquera

---

## What Is a Blueprint?

A pattern solves one problem. A blueprint shows how to build a whole capability by composing the right patterns together. You look at a blueprint and think "that's the thing I want to build."

---

## The Capability

Operational Cadence gives a hybrid human-AI team the ability to:

1. **Start each day with a unified plan** that everyone (human and AI) executes against
2. **Assign work through RACI** so ownership is structural, not assumed
3. **Surface blockers within hours**, not days or weeks
4. **Close the feedback loop at end of day** by reconciling what happened against the plan
5. **Control notification volume** so the right people see the right things

The result: a human can step away for a day and come back to a status report, not chaos.

---

## Patterns Composed

| # | Pattern | What It Contributes |
|---|---------|-------------------|
| 1 | **Runbook-Driven Agent Cadence** | Playbooks that define what work exists for each business event type. YAML runbooks with task templates, RACI, dependencies, and variable resolution. |
| 2 | **Plan of the Day** | Daily synthesis engine that reads runbooks + event calendars, deduplicates against in-flight work, and produces one executable plan with RACI assignments. |
| 3 | **RACI-Scoped Notifications** | Controls who sees what. Notification routing based on each person's RACI role per task. Prevents spam without missing critical updates. |
| 4 | **Escalation Chain with SLA** | When an agent is blocked, they raise it within an 8-hour window. Unraised blockers are tracked as the worst performance failure. |
| 5 | **EOD Reconciliation** | End-of-day sweep that cross-references open tasks against evidence sources (emails sent, CRM changes, calendar events). Closes implicitly-done tasks, flags true carryover. |

---

## How They Interact

```
Runbooks ──defines tasks──▶ Plan of the Day
                                │
                    ┌───────────┼───────────┐
                    │           │           │
                    ▼           ▼           ▼
              RACI-Scoped   Escalation   EOD
              Notifications   Chain    Reconciliation
                    │           │           │
                    └───────────┼───────────┘
                                │
                         ▼ feeds back to ▼
                      Next day's POD
```

**The feedback loop is what makes this more than a collection of patterns.** EOD Reconciliation closes tasks that got done, identifies ones that didn't, and surfaces unraised blockers. This feeds tomorrow's POD, which generates a cleaner plan because yesterday's noise has been cleared. Over time, the system self-corrects: tasks that keep carrying forward signal a runbook problem (wrong timeline, wrong owner, wrong dependencies).

---

## Emergent Properties

No individual pattern here is revolutionary. Runbooks exist everywhere. Daily standups exist everywhere. RACI charts exist everywhere. But the composition creates properties that no single pattern provides:

**Self-correcting execution.** The system detects its own gaps (EOD Reconciliation), surfaces its own blockers (Escalation Chain), and controls its own noise (RACI Notifications). It doesn't wait for a human to notice something is wrong.

**Structural accountability.** Every task has a named owner (R), a named accountable party (A), and named stakeholders (C, I). "I didn't know it was my job" is structurally impossible.

**Compounding quality.** Each day's EOD feeds the next day's POD. Each week's leaderboard feeds the monthly ladder evaluation. Each month's retrospective feeds the runbook updates. The system gets better at managing itself over time.

---

## When to Use This Blueprint

You need Operational Cadence when:

- You're running 3+ AI agents alongside humans
- Work comes from multiple recurring sources (events, releases, campaigns, reviews)
- You've had the "overdue task graveyard" problem (auto-generated tasks nobody closes)
- Your team starts each day asking "what should I work on?"
- Alert fatigue is real (too many notifications from too many agents)
- You want to step away for a day without things falling apart

You don't need this blueprint when:

- You have one agent doing one job (just give it a runbook)
- All work is ad hoc with no recurring patterns
- Your team is small enough that a Slack standup covers coordination

---

## Implementation Sequence

```
Week 1: Runbooks + Registry
  Write YAML runbooks for your top 2 event types
  Set up event registry (CRM table or YAML files)
  Validate: "cadence runbook simulate" produces sensible tasks

Week 2: Plan of the Day
  Build POD generator that reads runbooks + registry
  Wire to daily cron (6 AM generation, 7 AM delivery)
  Validate: POD produces the right tasks with the right owners

Week 3: Execution Loop
  Add task completion, escalation, carryover commands
  Wire agents to receive and execute POD tasks
  Add midday status check (12 PM)

Week 4: Feedback Loop
  Build EOD Reconciliation (9 PM evidence-based sweep)
  Add RACI-scoped notification routing
  Enable escalation SLAs
  Validate: yesterday's EOD feeds today's cleaner POD

Week 5+: Tune
  Add leaderboard and metrics (Ladder of Trust integration)
  Write additional runbooks
  Expand notification channels to team
```

---

## Known Implementation

**Cloud Nirvana AIOS (Cadence framework):**
- 11 AI agents + 7 human team members
- 5 runbook types (conference, podcast, newsletter, partnership, strategic planning)
- Daily POD delivered at 7 AM ET
- Midday status at 12 PM, EOD report at 9 PM
- Weekly leaderboard with Agent of the Week recognition
- Production since May 2026

---

## Origin

Built after discovering that auto-generated tasks without context, accountability, or feedback loops create noise instead of progress. 33 phantom overdue tasks in Notion proved that a task tracker isn't a project manager. Inspired by U.S. Navy Plan of the Day operations and Ben Blanquera's operating philosophy: "Get shit done" and "Alone we go fast, together we go far."

---

*Blueprint published May 2026 by Sean Erikson & Lou, Cloud Nirvana.*
*License: CC BY 4.0*
