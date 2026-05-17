# Checkpoint-Gated Autonomy

**Status:** Implemented  
**Category:** Trust & Governance  
**First Implemented:** Cloud Nirvana AIOS, May 17 2026  
**Source:** Real production implementation, not theoretical

---

## Problem

AI agents need human approval for high-stakes actions (sending emails, executing payments, publishing content), but keeping agent sessions alive while waiting for human response is expensive, fragile, and creates timeout failures. The agent either burns resources waiting, or the session dies and loses context.

## Forces

- Agents run as ephemeral sessions (cron jobs, subagent spawns) with finite timeouts
- Humans respond on their own schedule (minutes, hours, sometimes days)
- Approval context must survive across session boundaries
- Trust levels vary by agent AND by action type (an agent might be autonomous for headcount emails but require approval for partner communications)
- The system should improve over time; manual approval gates should eventually become unnecessary for proven action types

## Solution

**Decouple work from approval using durable checkpoints.** The agent produces an artifact (e.g., Gmail draft) and writes a checkpoint file containing everything needed to complete the action. The session ends. When the human approves, a new lightweight session reads the checkpoint and executes.

```
Agent Session                    Human                    Handler Session
─────────────                    ─────                    ───────────────
1. Verify data (SoR)
2. Create artifact
3. Run QA rules
4. Write checkpoint file
5. Present with buttons
6. Session ends
                                 7. Reviews
                                 8. Clicks Send/Edit
                                                          9. Read checkpoint
                                                          10. Execute action
                                                          11. Confirm to human
                                                          12. Archive checkpoint
```

### Key Design Decisions

**Checkpoint contains everything.** The handler session needs zero context beyond the checkpoint file. No session history, no memory, no conversation thread. The checkpoint IS the context.

**Buttons carry artifact IDs.** Inline buttons (Telegram, Slack, etc.) embed the artifact ID in the callback value (`agent_send_draftID`). Even if multiple drafts are pending, each button is self-contained.

**QA rules are structural, not advisory.** Quality rules are enforced inside the artifact creation MCP (the agent cannot create a non-compliant artifact), AND exposed to the agent as guidance before drafting. One source of truth, two consumers.

**Trust ladder determines gate existence.** The checkpoint gate is where autonomy lives:
- Mode 2: Agent creates artifact, checkpoint, presents with buttons, human approves
- Mode 3: Agent creates artifact, checkpoint, auto-executes (no human gate)

Trust promotion is per-action-type, not per-agent. An agent can be Mode 3 for routine actions and Mode 2 for sensitive ones.

## Resulting Context

**Benefits:**
- No timeout pressure (sessions end immediately after presenting)
- No context loss (checkpoint file is durable state)
- Edit flows work naturally (each round is a new stateless session)
- QA improves continuously (human feedback promotes to rules)
- Agents climb the trust ladder (proven agents skip the gate)
- Scales to any number of agents (same pattern, same checkpoint format)

**Tradeoffs:**
- Requires a persistent handler (e.g., a strategic agent or webhook) to process button clicks
- Checkpoint files need cleanup (archival after execution, periodic purge of stale checkpoints)
- Initial setup is more complex than a simple "wait for response" loop

## Related Patterns

- **Ladder of Trust:** Determines whether the checkpoint gate exists for a given agent + action type
- **Data Provenance Enforcement:** QA rules enforce that all data in artifacts comes from verified sources
- **Memory Flush Discipline:** Checkpoint files serve as durable memory across session boundaries

## Implementation Reference

Cloud Nirvana AIOS implementation:
- `mcp-server/approval-gate` — checkpoint CRUD, execute, edit, history
- `mcp-server/draft-present` — standardized presentation with buttons
- `mcp-server/qa-check` — self-improving QA rules engine
- `config/qa-rules.json` — growing rule set (feedback-driven)
- `pending-approvals/` — active checkpoints
- `pending-approvals/history/` — audit trail
- `docs/agent-onboarding/EMAIL-APPROVAL-PATTERN.md` — canonical agent workflow
