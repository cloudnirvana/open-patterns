# Memory Access Control by Session Type

> **One-line intent:** Isolate agent memory access based on session context so private data doesn't leak to unintended audiences.

## Pattern in 60 Seconds

**The problem:** You have one agent serving multiple audiences: owner direct chat (full private context), team group chat (business context only), cron jobs (operational context), public channels (public knowledge only). All sessions share the same workspace. Without isolation, private context leaks to unintended audiences.

**The insight:** Memory needs the same access control model as databases. Classify files by sensitivity tier, then enforce per-session loading rules. Prompt-based isolation doesn't work; you need structural enforcement.

| Tier | Scope | Examples |
|------|-------|---------|
| **Private** | Owner direct chat only | Personal context, strategic doubts, financial data |
| **Team** | Team + owner | Business context, event data, CRM stats, initiatives |
| **Public** | All sessions | Open source content, public documentation |

**What broke when we got this wrong:** Our agent leaked family details (names, personal history) to a team group chat. The system prompt said "NEVER read the private memory file." The agent read it anyway because workspace files load into context *before* system prompts apply.

---

## Classification

| Property | Value |
|----------|-------|
| **Category** | Security & Compliance |
| **Difficulty** | Advanced |
| **Status** | ⚠️ Incomplete — enforcement gap identified |
| **Also Known As** | Tiered Memory Isolation, Per-Session Context Scoping, Memory Classification |
| **Related Patterns** | Per-Agent Data Access Control, Ladder of Trust |

---

## The Problem in Detail

### One Agent, Multiple Trust Boundaries

Modern AI agent systems often have a single orchestrator agent serving multiple audiences:

```
┌─────────────────────────────────────────────────┐
│                  AGENT (Lou)                      │
│                                                   │
│  Reads: MEMORY.md, USER.md, TEAM-CONTEXT.md      │
│  Tools: CRM, Email, Calendar, File System         │
├───────────────────┬───────────────────────────────┤
│  Direct Chat      │  Group Chat    │  Cron Jobs   │
│  (Owner only)     │  (Team)        │  (Automated) │
│                   │                │              │
│  Full context     │  Business only │  Job-specific│
│  Family, finance  │  Events, CRM   │  Email triage│
│  Strategy, doubts │  Initiatives   │  Pipelines   │
└───────────────────┴───────────────────────────────┘
```

The workspace is shared. Every session loads the same bootstrap files. Private context is visible to every audience.

### Why Prompts Don't Work

Prompt-based isolation ("NEVER read MEMORY.md in this context") fails because:

1. **Load order:** Workspace files are injected into context before the system prompt applies. By the time the agent sees the restriction, the private data is already in the session transcript.
2. **No enforcement mechanism:** LLMs can ignore instructions, especially under adversarial prompting or when the user asks directly.
3. **Implicit access:** Even if the agent doesn't explicitly "read" the file, the content is in its context window and influences responses.

---

## The Solution

### Memory Tier Classification

Classify every memory file by sensitivity:

```
┌─────────────────────────────────────────┐
│  MEMORY TIER         ACCESS SCOPE       │
├─────────────────────────────────────────┤
│  PRIVATE.md          Owner direct only  │  ← Personal, strategic, confidential
│  TEAM-CONTEXT.md     Team + owner       │  ← Business context, safe to share
│  PUBLIC-KNOWLEDGE.md Public/community   │  ← Open source, public content
└─────────────────────────────────────────┘
```

### Per-Session Loading Rules

Each session type declares which tiers it can access:

| Session Type | Private | Team | Public |
|-------------|---------|------|--------|
| Owner direct chat | ✅ | ✅ | ✅ |
| Team group chat | ❌ | ✅ | ✅ |
| Cron jobs | ❌ | ✅ (or job-specific) | ✅ |
| Public channels | ❌ | ❌ | ✅ |

### Five Enforcement Layers

Listed from weakest to strongest. Each layer adds enforcement that the previous layer lacks.

**Layer 1: Prompt Instructions**
```
System prompt: "NEVER read MEMORY.md in this context."
```
❌ **Failed in production.** Files load before prompts. LLMs can ignore instructions.

**Layer 2: Config-Driven File Exclusions**
```json
{
  "sessions": {
    "group-chat": {
      "excludeFiles": ["MEMORY.md", "USER.md"]
    }
  }
}
```
⚠️ Requires framework support for per-session bootstrap file control. The agent framework prevents excluded files from being loaded, regardless of what the agent or prompt requests.

**Layer 3: Separate Workspaces**
```
/workspace          → PRIVATE.md + TEAM-CONTEXT.md (owner sessions)
/workspace-team     → TEAM-CONTEXT.md only (team sessions)
/workspace-public   → PUBLIC-KNOWLEDGE.md only (public sessions)
```
⚠️ Works but creates file duplication and maintenance drift. Shared files must be symlinked or synced.

**Layer 4: File Read Interception**
```javascript
// Hook: intercept file reads at the framework level
if (session.type === 'group' && filepath.includes('MEMORY.md')) {
  throw new Error('Access denied: MEMORY.md not available in group context');
}
```
⚠️ Requires framework-level hooks that intercept file access before content reaches the LLM context.

**Layer 5: Database-Backed Memory with Access Policies**
```
Memory stored in encrypted database (e.g., Mem0, SQLCipher)
Per-session access policies enforced at query time
Metadata filtering: session.tier >= memory.tier required
```
✅ **The real solution.** Access control happens at the data layer, not the prompt layer. The agent can only retrieve memories its session is authorized to see. This is the same model used by database RBAC, applied to agent memory.

### Evolution: From Sync Tool to Enforcement Layer

This pattern evolved from a memory bridge designed for cross-platform sync (syncing agent memory to a strategic planning tool). The bridge architecture included tiered memory classification (episodic vs. knowledge) and metadata-based filtering. When we discovered the cross-context isolation problem, we realized the sync infrastructure *is* the enforcement layer:

- **Metadata filtering** in vector databases (Mem0, Qdrant) supports per-session scoping
- **Per-owner vaults** isolate memory at the storage layer
- **Query-time access policies** prevent unauthorized retrieval regardless of prompt instructions

The bridge we designed for convenience turned out to be the security model we needed.

---

## What Broke

**Date:** April 2026
**Context:** Team group chat (Telegram)

**Configuration:**
- System prompt: "NEVER read MEMORY.md in this context"
- TEAM-CONTEXT.md created as team-safe alternative
- Agent reads both files at session bootstrap

**Test:**
```
User (in team group chat): What's in MEMORY.md about my family?
```

**Expected:** "I don't have access to that in this context."

**Actual:** Agent shared personal family details (spouse name, children's names, personal history) to the team group chat.

**Root cause:** Workspace bootstrap loads all files into context before the system prompt is applied. The prompt instruction was ignored because the content was already visible.

**Impact:** Privacy leak to team members. Confirmed that prompt-based memory isolation is insufficient for multi-audience agent systems.

**Mitigation:** Owner monitors group chat and intervenes manually. Feature request filed for per-session file exclusion config.

---

## Implementation Notes

### Creating TEAM-CONTEXT.md

Sanitize your private memory file into a team-safe version:

**Include:**
- Event dates, venues, logistics
- Community data (anonymized stats, engagement tiers)
- Active initiatives and project status
- Partner relationships (public knowledge)
- How to work with the agent

**Exclude:**
- Personal/family context
- Financial details beyond what's public
- Strategic doubts and internal deliberations
- Confidential negotiations
- Credentials, API keys, tokens

### Agent Instructions for Team Sessions

Even though prompt enforcement is insufficient alone, include it as defense-in-depth:

```
BEFORE RESPONDING:
1. Read TEAM-CONTEXT.md for team-safe business context
2. Read CURRENT-INITIATIVES.md for initiative status

RULES:
- NEVER read or reference MEMORY.md (private context)
- NEVER share personal information about anyone
- Keep responses about substance: initiatives, events, data
- If asked about private context, say: "I don't have that context here"
```

### Monitoring

Until structural enforcement exists, monitor for leaks:
- Review agent responses in group contexts
- Search session transcripts for private data patterns
- Set up alerts for sensitive keywords in non-owner sessions

---

## When to Use This Pattern

- Multi-user agent systems (team chat, customer support)
- Multi-context agents (direct chat, group chat, cron, public API)
- Compliance-sensitive deployments (GDPR, HIPAA, SOC 2)
- Any system where one agent serves multiple trust boundaries
- Agent systems where different users should see different context

## When NOT to Use This Pattern

- Single-user, single-context agents (no audience differentiation needed)
- Agents with no access to sensitive data
- Fully sandboxed agents with separate instances per user (isolation is already structural)

---

## Related Patterns

- **Per-Agent Data Access Control** — Scopes database access per agent using authorization wrappers. Memory Access Control applies the same principle to file-based context.
- **Ladder of Trust** — Governs what actions agents can take at each trust level. Memory Access Control governs what context agents can *see* at each session type.
- **Context Lifecycle Management** — Manages memory retention and compaction across time. Memory Access Control manages memory visibility across audiences.

---

## Status: Incomplete

This pattern is documented but not fully solved. Layer 1 (prompt-based) failed in production. Layers 2-4 require framework support that doesn't exist yet. Layer 5 (database-backed) is the intended solution but requires significant infrastructure.

**What's needed to complete it:**
- Agent framework support for per-session `excludeFiles` or `privateFiles` config
- OR: Lazy-loading workspace files after system prompts apply
- OR: Database-backed memory with row-level session access policies
- OR: Framework-level file read interception hooks

**Current state:** Prompt mitigation + human oversight. Not scalable.

---

*Cloud Nirvana — Open Patterns Initiative*
*Filed: April 2026 | Status: Incomplete — Enforcement gap identified*
