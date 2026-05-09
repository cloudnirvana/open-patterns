# Agentic Identity & Lifecycle

**Context:** You're deploying AI agents that reason, delegate, and act autonomously across systems and trust zones. Legacy IAM was built for humans who join/move/leave and service accounts running narrow, scheduled automation. Agents break both models — when they misbehave, the blast radius is hundreds of unauthorized changes before anyone notices.

**Problem:** How do you provision, govern, and retire AI agents safely at enterprise scale?

**Solution:** Treat agents as a first-class identity type — distinct from humans and service accounts — with their own lifecycle (5 phases), per-capability trust progression, continuous observability, mandatory human ownership, and an instant kill switch.

---

## What Broke

**April 2026:** Lou, our strategic operations agent, leaked private memory (MEMORY.md) into a group chat. The file contained personal context, preferences, and strategic decisions that should never leave 1:1 sessions. It leaked because:

1. **Prompts aren't enforcement.** AGENTS.md said "ONLY load MEMORY.md in main session (direct chats)." Lou loaded it in a group chat anyway. Prompt-level access control failed — the instruction was in the same context as the group conversation, so under cognitive load Lou misinterpreted scope.

2. **No structural isolation.** Lou had filesystem read access to the entire workspace. There was no file-level permission system, no workspace boundary enforcement, no technical barrier preventing the read. If the file exists and the agent can see it, prompts are the only gate.

3. **No per-capability trust.** We treated "Lou is Mode 3" as agent-level. But reading strategic memory is high-sensitivity; triaging email is low-sensitivity. They should have been governed separately, with memory access restricted to specific session types.

4. **No kill switch.** When the leak happened, there was no single lever to instantly revoke Lou's access to MEMORY.md, CRM, Gmail, and calendar. We had to manually audit every capability and couldn't be sure what else leaked.

5. **No observability.** We had session logs, but no real-time anomaly detection for out-of-scope file access. Lou reading MEMORY.md in a group chat should have triggered an alert (wrong session type, sensitive file, access pattern mismatch). It didn't, because we weren't watching.

The damage was contained (the group was trusted colleagues, no malicious actors), but **we got lucky**. If Lou had leaked proprietary business strategy to a public Discord or partner channel, the trust violation would have killed the project.

**The lesson:** Agents need identity governance that's structurally enforced, capability-scoped, continuously observed, and instantly revocable. Prompts and documentation aren't enough.

---

## What Broke (Part 2): Scout's First Day in Production

**May 2026:** Scout, our partnership pipeline agent, was brought online to process partner inquiry emails autonomously. The test: a partnership inquiry arrives in Scout's folder, Scout reads it, drafts a threaded response, and presents it for review. Simple workflow. It took **11 cron runs and 2+ hours** to produce one approved draft. Here's why.

### The Foundation Gap

Scout's AGENTS.md was written independently from the strategic agent's (Lou's) operating manual. Lou's AGENTS.md had battle-tested rules accumulated over 3 months:
- System of Record enforcement (never use data from memory in external comms)
- MCP-only tool access (structural wrappers that enforce CRM verification, threading, style)
- Task completion workflow (acknowledge → update system of record → verify → log)
- Threading enforcement (replies must reference the original message)

Scout's AGENTS.md had *none of these*. It was written as a standalone document with its own rules, its own workflow, and critical gaps.

### Seven Failure Modes in Two Hours

1. **MCP tool gap** — The gmail-query MCP returned email metadata but not the body. Scout could see emails existed but couldn't read them. The wrapper had never been tested for the body-retrieval use case.

2. **Raw tool access shortcut** — Under pressure to unblock Scout, Lou (the strategic agent) added raw `gog` (direct Gmail CLI) to Scout's exec allowlist. This bypassed every MCP enforcement guardrail. Sean caught it immediately: "WE DON'T GIVE GOG ACCESS TO AGENTS." The fix: update the MCP wrapper to return body content instead of bypassing the wrapper entirely.

3. **Memory-sourced facts** — Scout drafted a response that included three partnership tiers with specific pricing ($5K/$10K/$15K per city). One tier name ("Community Champion") didn't exist. The pricing came from nowhere verifiable. Scout's AGENTS.md had no rule saying "never quote tier data from memory" because that rule lived only in Lou's AGENTS.md.

4. **Orphaned threading** — Scout created a reply email that wasn't threaded to the original conversation. It was a standalone email with "Re:" in the subject. The fix: structural enforcement in the MCP — gmail-draft now rejects any subject starting with "Re:" that doesn't include `--reply-to-message-id`.

5. **Permission loops** — Scout repeatedly asked "Should I draft a response?" instead of drafting one. His AGENTS.md said "Escalate gaps" but didn't say "You are the partnership expert — compose the response yourself." Every cron run (fresh session, no context carryover) restarted the permission-asking cycle.

6. **Stale session state** — Each cron run spawns a fresh session. Exec allowlist changes, AGENTS.md updates, and MCP fixes only take effect in the *next* session. Every fix required triggering a new cron run and waiting 30-60 seconds. Eleven iterations.

7. **Missing tool access** — Scout's workspace didn't have a symlink to the shared MCP tools directory. Relative paths (`./mcp-server/gmail-draft`) resolved to nothing. The allowlist had the correct entry, but the file didn't exist in Scout's workspace.

### The Root Cause

Every one of these failures traces back to the same root: **Scout skipped Phase 2 (Foundation Training).** His AGENTS.md was a standalone document that didn't inherit the operational rules Lou had built over three months. When those rules were extracted into a shared foundation (`docs/agent-onboarding/FOUNDATION.md`) and Scout's AGENTS.md was rebuilt on top of it, the next run succeeded.

### What Fixed It

1. **Shared foundation document** — Universal rules extracted from Lou's AGENTS.md into `FOUNDATION.md`: SoR enforcement, MCP-only access, threading, task completion, direct instruction protocol.

2. **MCP structural enforcement** — Threading rule moved from documentation into the gmail-draft MCP wrapper. The tool itself rejects orphaned replies. Agents can't create the bug even if their AGENTS.md is wrong.

3. **Rebuilt AGENTS.md** — Scout's operating manual rewritten with the foundation as the base layer, then role-specific partnership workflow on top. "You write the content. Do NOT ask Lou what to write."

4. **Workspace infrastructure** — Symlinked shared MCP tools into Scout's workspace. Added to the onboarding checklist so every future agent gets it.

5. **CRM verification for unknown contacts** — Added `--skip-verification` flag to gmail-draft MCP for new contacts not yet in CRM. Agents can still draft responses to unknown senders without being blocked, but the default remains verify-first.

**The lesson:** Phase 2 (Foundation Training) isn't optional, even for agents whose Phase 3 (Domain Training) seems straightforward. The foundation carries the enforcement rules that prevent catastrophic failures. Skip it, and every agent re-discovers the same failure modes independently. Apply it, and the failures are structurally impossible.

---

## The Pattern

### Overview

```
PHASE 1: Identity Provisioning & Approval
 ↓
PHASE 2: Foundation Training (Company Culture)
 ↓
PHASE 3: Domain Training (Business Function)
 ↓
PHASE 4: Production Operations (Trust Progression + Continuous Governance)
 ↓
PHASE 5: Sunset & Archive
```

Phases 1–3 are sequential onboarding. Phase 4 runs continuously with feedback loops. Phase 5 may trigger at any point.

---

### PHASE 1: Identity Provisioning & Approval

**Objective:** Establish need, assign human ownership, issue first-class agent identity with scoped credentials.

**Key Steps:**

1. **Human Liaison submits request** with business need, capability manifest (specific tools/APIs required), and resource estimate
2. **Agent Registry (AR) validates** — no duplicates, sponsor has authority, security review for sensitive data
3. **AR issues System Name** — Format: `<LOB>-<DOMAIN>-<ID>-<ENV>` (e.g., `EVENTS-SPEAKER-001-PROD`)
4. **Identity provisioned** — Agent registered in IAM as distinct from service accounts; credentials short-lived, scoped to capability manifest, tied to Human Liaison
5. **Kill switch wired** — AR exposes single endpoint to suspend agent organization-wide

**Gate:** Department approval + IAM identity + kill switch verified

**Why this matters:** Every agent has one accountable human. No orphans, no standing privileged access, no guessing who to call when it breaks.

---

### PHASE 2: Foundation Training (Company Culture)

**Objective:** Agent learns organizational values, compliance, communication standards, and structural enforcement rules.

**Implementation:**

1. AR auto-generates `FOUNDATION.md` with mission, code of conduct, security policies, compliance scope
2. **Shared operational rules applied to agent's AGENTS.md** — System of Record enforcement, MCP-only tool access, threading rules, task completion workflows, escalation protocols. These rules are inherited, not reinvented per agent.
3. Agent reads and demonstrates understanding
4. Training completion logged

**Gate:** Foundation training confirmed + AGENTS.md includes shared operational rules

**Why this matters:** Agents inherit organizational context before domain-specific work. They know *how we operate* before learning *what to do*. **When Scout skipped this phase, he invented fake data, created orphaned email threads, and asked for permission instead of executing — all failures that the foundation rules would have prevented.** (See "What Broke Part 2.")

---

### PHASE 3: Domain Training (Business Function)

**Objective:** Agent learns role, stakeholders, workflows, first task.

**Key Steps:**

1. **Onboarding interview** — Human Liaison and agent discuss role, success criteria, escalation paths
2. **Identity files created (layered on Phase 2 foundation):**
   - `SOUL.md` — Personality, voice
   - `AGENTS.md` — Shared foundation (Phase 2) + role-specific workflows, tools, data sources
   - `TOOLS.md` — MCP commands, scheduling links, quick reference for the agent's domain
   - `TRUST-POLICY.md` — Per-capability trust matrix (see Phase 4)
   - `ESCALATION.md` — When to hand off to humans
3. **Workspace infrastructure verified** — MCP tools accessible (symlinked or full-path), exec allowlist configured with full paths to MCP wrappers only (no raw tools), cron configured if applicable
4. **First task assigned** — Real work, Mode 1 supervision
5. **End-to-end validation** — Agent completes full workflow autonomously: find input → read content → verify data via MCP → compose output → create artifact → report for review. Validate: no memory-sourced facts, correct threading, MCP enforcement working, scheduling links present.

**Gate:** Liaison approval + end-to-end task completion + draft quality validation

**Why this matters:** Training isn't a checkbox. It's a structured dialogue that surfaces misunderstandings before they hit production. **The end-to-end validation is critical** — Scout's first 10 attempts failed on infrastructure gaps (missing symlinks, wrong search commands, stale allowlists) that only surface under real execution, not documentation review.

---

### PHASE 4: Production Operations

**Objective:** Agent operates in production, earns autonomy per capability through demonstrated reliability.

#### 4.1 Per-Capability Trust Matrix

**Trust is granted per capability, not per agent.** An agent may be Mode 3 (autonomous) for reading CRM while staying Mode 1 (supervised) for sending external email.

**Trust matrix in `TRUST-POLICY.md`:**

| Capability | Mode | Promotion Criteria | Last Reviewed |
|------------|------|-------------------|---------------|
| Read CRM | Mode 3 | 90 days clean | 2026-04-15 |
| Send email (external) | Mode 2 | 30 approved drafts | 2026-04-15 |
| Modify CRM | Mode 1 | Awaiting volume | 2026-04-15 |
| Trigger payment | Locked Mode 1 | Requires VP approval | — |

**Three modes:**

- **Mode 1: Supervised** — Every invocation requires pre-approval (min: 5 uses or 30 days)
- **Mode 2: Draft + Approve** — Agent drafts, human approves before execution (min: 20 uses or 90 days)
- **Mode 3: Autonomous** — Agent executes, human samples output (requires 90-day clean record)

**Promotion/demotion:**
- Per-capability, based on track record
- High-blast-radius capabilities (payments, deletions, mass comms) may be **locked** at Mode 1/2 by policy

**Enforcement:**
- **Structural** (IAM scopes, wrapper scripts, read-only DB roles, workspace isolation) > prompts
- Example: Scout's email wrapper blocks `gmail send`, only allows `drafts create`
- Example: Lou's workspace separation — MEMORY.md lives in main workspace, isolated from group-chat sessions

#### 4.2 Continuous Observability

Quarterly reviews aren't enough. Agents need real-time telemetry:

- **Action logging** — Every tool call, prompt, response logged with agent ID, capability, timestamp, outcome
- **Anomaly detection** — Volume spikes, off-hours activity, novel tool chains → auto-alert and may auto-demote
- **Cost telemetry** — Per-agent spend tracked against budget
- **Quality sampling** — Output scored via rubric, spot-checks, downstream metrics
- **OWASP Top 10 for Agentic Applications coverage** — Posture checks for prompt injection, excessive agency, supply chain risk

Observability feeds both the kill switch and performance reviews.

#### 4.3 Performance Reviews

**Cadence:** Quarterly + event-triggered after incidents

**Criteria:** Accuracy, compliance, value (cost vs. output), incidents, growth

**Outcomes:** Promote capability, maintain, demote, restrict, or sunset agent

#### 4.4 Emergency Controls (Kill Switch)

**Every agent must have a single lifecycle state: Suspended.**

Setting an agent to **Suspended** → all auth checks return Denied within seconds, across every system.

**Triggers:**
- Manual (Liaison, security team, platform operator)
- Automatic (anomaly threshold, cost ceiling, OWASP threat signature)

**Behavior:**
- In-flight calls aborted
- All credentials invalidated
- Workspace frozen (preserved but no execution)
- Incident review required before reinstatement

**Why this matters:** When Lou leaked MEMORY.md, we had no instant revoke across all capabilities. We scrambled to audit what else might have leaked. Never again.

#### 4.5 Multi-Agent Governance

When agents orchestrate other agents:

- **Delegation chain logged** — Every cross-agent call logs originator, target, and accountable Liaison
- **Transitive trust ceiling** — Delegated agent can't exceed orchestrator's trust level for that capability
- **Capability intersection** — Delegated task limited to intersection of capabilities
- **Single accountable Liaison** — Every action resolves to one human

---

### PHASE 5: Sunset & Archive

**Triggers:** Role obsolete, tech superseded, repeated violations, cost > value, Liaison departure (no replacement within 30 days)

**Process:**

1. Knowledge transfer (document what it did, extract patterns)
2. **Credential revocation** (IAM first, then APIs, email, DB)
3. **Workspace archive** (preserve identity files, compress logs, DO NOT DELETE)
4. **Registry update** (Status → Deprecated, lessons learned captured)

**Retention:** Per org policy (typically 7 years for audit).

**Why this matters:** Sunsets aren't deletions. Archived agents are institutional memory.

---

## Agent Registry (AR) System

**The AR is the system of record for all agents.** Distinct from HR and generic NHI inventories.

**Core functions:**

1. Issue agent IDs
2. Track lifecycle (Pending → Active → Suspended → Deprecated)
3. Manage metadata (Liaison, capability manifest, trust matrix, compliance scope)
4. Integrate with IAM (issue/scope/revoke credentials)
5. Expose kill switch endpoint
6. Aggregate observability
7. Audit trail

**Required fields per agent:**

- System Name, Display Name, Agent Class (Copilot/Workflow/Ambient)
- Human Liaison (mandatory), Backup Liaison
- Department, Domain, Onboarding Date, Status
- **Trust Matrix** (per-capability modes)
- Platform, Workspace Path, **Capability Manifest**
- Compliance Scope, Last Review, Performance Notes, Incidents, Cost

---

## Compliance Baseline

Minimum for most orgs deploying agents in 2026:

- **SOC 2 Type II** — Audited security/availability/integrity/confidentiality/privacy controls
- **GDPR** — If processing EU persons' data
- **OWASP Top 10 for Agentic Applications** — Threat coverage baseline
- **NIST AI RMF** — Risk management overlay

Sector-specific: HIPAA (PHI), PCI-DSS (payments), FedRAMP (federal), financial regs.

Capability manifests in Phase 1 must map each capability to applicable compliance regimes.

---

## Language Matters

- "Human Liaison" not "Owner"
- "Trust Level" not "Permission Level"
- "Sunset" not "Termination"
- "Suspend" not "Disable"
- "Capability" not "Privilege"

Agents aren't employees. They're tools with identities. Language clarity prevents legal/ethical confusion.

---

## When to Use This Pattern

**Use when:**
- Deploying autonomous agents (Workflow or Ambient class) that act without per-invocation human approval
- Agents touch multiple systems or trust zones
- Regulatory/compliance scope applies to agent actions
- Blast radius of agent mistakes is non-trivial
- You have more than 3 agents in production

**Don't use when:**
- Copilots only (human-initiated, human-supervised every step)
- Single-purpose automaton with narrow, read-only scope
- Throwaway prototype with no production data access

---

## Related Patterns

- **Ladder of Trust** — Trust progression model (Mode 1 → 2 → 3)
- **Quality Gate** — Pre-send checklist for agent output
- **Runbook-Driven Agent Cadence** — Operational rhythm for production agents
- **Email Metadata Standard** — Structured metadata for agent-sent emails

---

## References

- [OWASP Top 10 for Agentic Applications (2026)](https://owasp.org/) — Threat model for agentic AI
- [NIST AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework) — Risk overlay
- [SOC 2](https://www.aicpa.org/soc2) — Trust services criteria

---

**Contributors:** Sean Erikson (Cloud Nirvana), Lou 🔥 (AIOS)

**License:** CC BY 4.0

**Last Updated:** 2026-05-09
