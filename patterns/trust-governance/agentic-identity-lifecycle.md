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

**Objective:** Agent learns organizational values, compliance, communication standards.

**Implementation:**

1. AR auto-generates `FOUNDATION.md` with mission, code of conduct, security policies, compliance scope
2. Agent reads and demonstrates understanding
3. Training completion logged

**Gate:** Foundation training confirmed

**Why this matters:** Agents inherit organizational context before domain-specific work. They know *how we operate* before learning *what to do*.

---

### PHASE 3: Domain Training (Business Function)

**Objective:** Agent learns role, stakeholders, workflows, first task.

**Key Steps:**

1. **Onboarding interview** — Human Liaison and agent discuss role, success criteria, escalation paths
2. **Identity files created:**
   - `SOUL.md` — Personality, voice
   - `AGENTS.md` — Role, workflows, tools
   - `TRUST-POLICY.md` — Per-capability trust matrix (see Phase 4)
   - `ESCALATION.md` — When to hand off to humans
3. **First task assigned** — Real work, Mode 1 supervision

**Gate:** Liaison approval + task completion

**Why this matters:** Training isn't a checkbox. It's a structured dialogue that surfaces misunderstandings before they hit production.

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

**Last Updated:** 2026-04-23
