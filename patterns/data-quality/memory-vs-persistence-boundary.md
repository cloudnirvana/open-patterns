# Memory vs Persistence Boundary

> **One-line intent:** Use a decision framework to determine when information belongs in agent memory (markdown files, session context) versus structured persistence (database).

## Pattern in 60 Seconds

**The problem:** An agent processes information all day. Some of it should be remembered forever (partner contracts). Some expires when the session ends (emotional context). Where do you draw the line?

**The insight:** The boundary isn't about importance; it's about structure, queryability, and reusability.

**The decision framework:** Six criteria determine memory vs persistence:

| Criterion | Memory (Files) | Persistence (Database) |
|-----------|----------------|------------------------|
| **Queryability** | Rarely searched; grepped when needed | Queried frequently across records |
| **Relationships** | Standalone; no foreign keys | Has relationships to other entities |
| **Multi-agent access** | One agent reads; hub coordinates | Multiple agents query concurrently |
| **Change frequency** | Changes rarely or narrative | Updates frequently, structured |
| **Fact vs narrative** | Narrative, emotional, contextual | Factual, structured, canonical |
| **One-off vs reusable** | Unique to this moment | Reused across sessions/agents |

**What broke when we got this wrong:** We stored sponsor data in a markdown file (`SPONSORS.md`). Multiple agents read and updated it. Within a week: duplicate entries, stale data, no referential integrity. One agent thought Sponsor A was "confirmed"; another thought "pending." No single source of truth.

**What survived:** CRM in SQLite (now SQLCipher) as system of record. Memory files for narrative context ("Sean was frustrated about Glean"). The boundary is clear: database for facts, memory for narrative.

---

## Classification

| Property | Value |
|----------|-------|
| **Category** | Data Quality |
| **Difficulty** | Intermediate |
| **Also Known As** | Ephemeral vs Durable Data, Memory vs Records, Session Context vs System of Record |

---

## Motivation

Your agent just finished a conversation with the user. The session contained:
- "Partner X signed the contract for $50K annually"
- "Event is scheduled for March 26, 2-6 PM"
- "Sean was frustrated that Glean's indexing missed key documents"
- "Decided to prioritize Cincinnati expansion over Cleveland"

Where do these go?

If everything goes in memory files, you lose queryability. How do you find all partners with annual contracts over $25K? How do you see all events in Q2? You're grepping markdown files.

If everything goes in a database, you lose context and narrative. "Sean was frustrated" doesn't fit in a `sentiment` ENUM. "Decided to prioritize" is a decision point, not a fact.

The Memory vs Persistence Boundary pattern provides a decision framework. It's not about what's important (everything is important). It's about structure, relationships, and how the data will be used.

---

## Applicability

Use this pattern when:
- Agents process both structured data (names, dates, amounts) and unstructured context (decisions, emotions, narrative)
- Multiple agents need access to the same data (system-of-record candidates)
- You need to query across records (e.g., "all events in Q2")
- Data has relationships (partners → contracts → invoices)
- You're deciding where new data should live

Do NOT use this pattern when:
- All your data is clearly structured (just use a database)
- All your data is narrative (just use memory files)
- You're working with a single agent that never queries across sessions (memory is fine)

---

## Structure

### The Boundary

```mermaid
graph TD
    subgraph "Session Context (Memory)"
        M1[MEMORY.md\nNarrative, decisions, context]
        M2[Daily logs\n2026-03-24.md]
        M3[Emotional context\n"Sean was frustrated"]
        M4[Decision points\n"Decided to prioritize Cincinnati"]
    end
    
    subgraph "System of Record (Database)"
        D1[(CRM\nContacts, companies, events)]
        D2[(Finance\nContracts, invoices, payments)]
        D3[(Analytics\nMetrics, attendance, engagement)]
    end
    
    AGENT[Agent Processing] --> DECISION{Decision Framework}
    
    DECISION -->|Narrative, one-off, emotional| M1
    DECISION -->|Queryable, structured, multi-agent| D1
    DECISION -->|Has relationships, changes frequently| D2
    
    style M1 fill:#fff3e0
    style M2 fill:#fff3e0
    style M3 fill:#fff3e0
    style M4 fill:#fff3e0
    style D1 fill:#e8f5e9
    style D2 fill:#e8f5e9
    style D3 fill:#e8f5e9
```

---

## Participants

| Participant | Role | Example |
|------------|------|---------|
| **Session Context (Memory)** | Narrative, decisions, emotional state; expires or fades over time | MEMORY.md, daily logs, session notes |
| **System of Record (Database)** | Structured facts with relationships; canonical source of truth | CRM (contacts, events), Finance (contracts, invoices) |
| **Agent** | Processes data and decides where it belongs using the framework | Any agent writing data during a session |
| **Hub Agent** | May consolidate memory across agents; enforces the boundary | Lou (hub) reviewing daily logs and extracting facts for database |

---

## How It Works

### Decision Framework (6 Criteria)

#### 1. Queryability
- **Memory (Files):** Data is rarely searched. When needed, grep or skim.
- **Database:** Data is queried frequently across many records.

**Example:**
- "Sean was frustrated about Glean" → Memory (grepped when needed)
- Event times (2-6 PM) → Database (queried for scheduling conflicts)

#### 2. Relationships (Foreign Keys)
- **Memory (Files):** Standalone. No links to other entities.
- **Database:** Has relationships to other tables (foreign keys).

**Example:**
- "Decided to prioritize Cincinnati expansion" → Memory (standalone decision)
- Partner contract → Database (links to partner_id, invoice_id)

#### 3. Multi-Agent Access
- **Memory (Files):** One agent reads. Hub coordinates if needed.
- **Database:** Multiple agents query concurrently.

**Example:**
- Daily notes → Memory (agent's own session)
- Partner tier status → Database (multiple agents check tier for access rules)

#### 4. Change Frequency
- **Memory (Files):** Changes rarely. Narrative doesn't update.
- **Database:** Updates frequently. Structured mutations.

**Example:**
- "Cincinnati expansion prioritized on 2026-03-24" → Memory (moment in time)
- Event attendance count → Database (increments as attendees register)

#### 5. Fact vs Narrative
- **Memory (Files):** Narrative, emotional, contextual. How it felt.
- **Database:** Factual, structured, canonical. What happened.

**Example:**
- "Sean was frustrated" → Memory (emotional context)
- Event name, date, venue → Database (facts)

#### 6. One-Off vs Reusable
- **Memory (Files):** Unique to this moment. Not queried later.
- **Database:** Reused across sessions and agents.

**Example:**
- "Decided to focus on practitioner-first messaging this quarter" → Memory (context for this quarter)
- Partner name, tier, contract amount → Database (referenced by multiple agents over time)

### Decision Table

Use this table when deciding where new data should live:

| Data Example | Queryable? | Relationships? | Multi-agent? | Changes often? | Fact or narrative? | Reusable? | **Decision** |
|--------------|-----------|----------------|--------------|----------------|-------------------|-----------|-------------|
| Event times (2-6 PM) | ✅ Yes | ✅ Yes (event_id) | ✅ Yes | ❌ No | ✅ Fact | ✅ Yes | **Database** |
| Partner financials | ✅ Yes | ✅ Yes (partner_id) | ✅ Yes | ✅ Yes | ✅ Fact | ✅ Yes | **Database** |
| "Sean frustrated about Glean" | ❌ No | ❌ No | ❌ No | ❌ No | ❌ Narrative | ❌ No | **Memory** |
| Daily decisions/notes | ❌ No | ❌ No | ❌ No | ❌ No | ❌ Narrative | ❌ No | **Memory** |
| Sponsor data (name, tier, status) | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Fact | ✅ Yes | **Database** |
| "Decided to prioritize Cincinnati" | ❌ No | ❌ No | ❌ No | ❌ No | ❌ Narrative | ❌ No | **Memory** |

**Rule of thumb:** If 3+ criteria point toward Database, it belongs in structured storage. If most point toward Memory, it belongs in files.

### Code Example (Agent Decision Logic)

```python
def classify_data(data):
    """
    Determine if data belongs in memory or database.
    Returns: "memory" or "database"
    """
    score = 0
    
    # Queryability: will we search for this across many records?
    if data.get("queryable"):
        score += 1
    
    # Relationships: does it link to other entities?
    if data.get("has_foreign_keys"):
        score += 1
    
    # Multi-agent: do multiple agents need this?
    if data.get("multi_agent_access"):
        score += 1
    
    # Change frequency: does it update often?
    if data.get("updates_frequently"):
        score += 1
    
    # Fact vs narrative: is it structured data?
    if data.get("is_fact"):
        score += 1
    
    # Reusability: used across sessions?
    if data.get("reusable"):
        score += 1
    
    # If 3+ criteria favor database, use database
    if score >= 3:
        return "database"
    else:
        return "memory"

# Example usage
event_data = {
    "content": "Event on March 26, 2-6 PM",
    "queryable": True,           # Need to check scheduling conflicts
    "has_foreign_keys": True,    # Links to event_id, venue_id
    "multi_agent_access": True,  # Multiple agents check calendar
    "updates_frequently": False, # Event time rarely changes
    "is_fact": True,             # Structured time data
    "reusable": True             # Referenced across sessions
}

decision = classify_data(event_data)  # Returns "database"
```

### Implementation Pattern

```python
# When agent processes session data:

session_data = [
    {"text": "Partner X signed contract for $50K", "type": "fact"},
    {"text": "Event March 26, 2-6 PM", "type": "fact"},
    {"text": "Sean frustrated about Glean", "type": "narrative"},
    {"text": "Prioritize Cincinnati expansion", "type": "decision"}
]

for item in session_data:
    classification = classify_data(item)
    
    if classification == "database":
        # Extract structured data, write to DB
        write_to_database(item)
    else:
        # Append to memory file
        append_to_memory(item)
```

---

## Consequences

### Benefits
- **Clear boundary.** Teams know where data lives. No "I think we stored that somewhere?" conversations.
- **Queryable when needed.** System-of-record data is in a database where SQL works. Memory data doesn't clog queries.
- **Narrative preserved.** Emotional context and decisions stay in human-readable files, not shoehorned into database columns.
- **Reduced schema churn.** You don't add a `frustration_level` column to the database just to capture narrative.
- **Multi-agent coordination.** Agents can query the database concurrently. Memory files stay isolated per agent.

### Liabilities
- **Judgment calls.** Some data sits on the boundary. Is "Sean prefers email over Slack" a fact (database) or context (memory)? You have to decide.
- **Data duplication.** Sometimes the same information exists in both places: "Partner X signed contract" in MEMORY.md (narrative) and `contracts` table (fact). This is intentional but can feel redundant.
- **Migration pain.** If you realize data should have been in a database from the start, migrating from markdown to structured storage is manual work.

### What Broke in Practice

**Sponsor data in markdown:** We stored sponsor details in `SPONSORS.md`:

```markdown
## Current Sponsors

- **Company A:** Gold tier, $10K, confirmed
- **Company B:** Silver tier, $5K, pending approval
- **Company C:** Gold tier, $10K, confirmed
```

Multiple agents read and updated this file. Within a week:
- **Duplicate entries:** Company A appeared twice with different amounts
- **Stale data:** One agent thought Company B was "pending," another thought "confirmed"
- **No referential integrity:** When a contact changed companies, their sponsor link was orphaned
- **No queryability:** "Show me all Gold tier sponsors" required parsing markdown

**Root cause:** Sponsor data had all six database criteria:
1. Queryable (needed to filter by tier)
2. Relationships (linked to contact_id, contract_id)
3. Multi-agent access (partnership agent, finance agent, event agent all read it)
4. Changes frequently (tier upgrades, renewals)
5. Fact-based (structured data: name, tier, amount)
6. Reusable (referenced across multiple sessions)

**Fix:** Migrated to `partner_contracts` table in encrypted finance database. Markdown file became narrative context only: "Company A expressed concern about logo placement on website."

**Calendar assumptions:** Agent drafted meeting responses using event times it couldn't verify (no calendar API access yet). It wrote: "I'm available at 2 PM on March 26" without checking.

**Root cause:** Event times are facts (database). Agent was treating them as narrative (memory).

**Fix:** Event times moved to `events` table. Agents query before making availability statements. Memory files capture "Sean prefers afternoon meetings" (narrative).

### What Survived in Practice

**CRM as system of record:** Contact names, companies, event details, attendance → all in encrypted SQLite (now SQLCipher). Multiple agents query concurrently. Foreign keys enforce relationships. SQL handles "show me all events in Q2."

**Memory files for narrative context:** 
- `MEMORY.md`: "Sean was frustrated that Glean's indexing missed key documents"
- Daily logs: "2026-03-24: Decided to prioritize Cincinnati expansion"
- Session notes: "Tracy mentioned concerns about sponsor engagement"

When agents need to understand tone, emotion, or decision-making context, they read memory files. When they need facts, they query the database.

**The boundary held:** Zero data corruption in the database. Zero cross-agent contamination in memory files. Clear separation of concerns.

---

## Implementation Notes

### Variations

**Hybrid Approach:** Some data starts in memory and migrates to database. Example: daily event notes → memory during the event → facts extracted to database afterward.

**Memory File Extraction:** Hub agent periodically reviews memory files and extracts facts for database. Acts as a data quality gate.

**Database + Annotations:** Store facts in database, narrative in a `notes` TEXT column. Example: `events` table has `start_time` (fact) and `notes` (narrative). This works for small-scale systems.

### Common Pitfalls

- **Treating everything as narrative.** If multiple agents need it or you'll query it later, it's a database candidate even if it feels ephemeral right now.
- **Over-structuring.** "Sean was frustrated" doesn't need a `sentiment` ENUM. Leave narrative in memory.
- **Ignoring change frequency.** Data that updates often (attendance counts, status flags) belongs in a database even if it starts as memory.
- **Not migrating when needed.** If you realize data should be in a database, migrate early. The longer you wait, the more markdown-based workarounds pile up.

### Migration Checklist (Memory → Database)

When you realize data should have been in a database:

1. **Extract unique values** from markdown files (grep, awk, manual review)
2. **Define schema** (tables, columns, foreign keys)
3. **Migrate data** (Python script, SQL INSERT statements)
4. **Update agents** to query database instead of reading files
5. **Archive old markdown files** (don't delete; keep for audit)
6. **Document the boundary** (update agent instructions)

---

## Security Implications

### Attack Surface
- **Memory files are plaintext.** Emotional context and decisions are readable if an attacker accesses the file system. This is usually acceptable (low sensitivity), but be aware.
- **Databases can be encrypted.** SQLCipher, TDE (Transparent Data Encryption) protect facts at rest. Memory files typically are not encrypted.
- **Boundary violations.** If an agent writes sensitive facts to memory files instead of the database, those facts bypass encryption and access control.

### Data Sensitivity
- **Facts are often sensitive.** Partner contract amounts, event attendance, financial data → these belong in encrypted databases.
- **Narrative can be sensitive.** "Sean disagreed with the board about funding strategy" is operationally sensitive. Treat memory files accordingly.
- **Logs reveal operational patterns.** Daily logs show what the system prioritized, who it contacted, what decisions were made. This is intelligence.

### Failure Modes
- **Data in wrong layer.** Sensitive facts in memory files → no encryption, no access control, no audit trail.
- **Stale data in memory.** If facts are duplicated in memory and database, memory can become stale. Database is authoritative; memory is reference.
- **Unauthorized database access.** If memory files contain database credentials or connection strings, an attacker can pivot from memory to database.

### Mitigations
- **Encrypt databases.** SQLCipher for SQLite, TDE for PostgreSQL/MySQL, encryption at rest for cloud databases.
- **Access control on memory files.** File permissions (chmod 700), per-user workspaces, encrypted file systems.
- **Audit boundary violations.** Log when agents write to memory vs database. Detect patterns (agent always writes facts to memory → training issue).
- **No credentials in memory.** Database passphrases live in credential stores (Keychain, Vault), not in memory files.

---

## Known Uses

| Organization | Context | Scale |
|-------------|---------|-------|
| Cloud Nirvana | Multi-agent system with CRM (database) + memory files per agent | Small team, 2,000+ community members |
| [Your implementation here] | [How you applied this pattern] | [Your scale] |

---

## Related Patterns

| Pattern | Relationship |
|---------|-------------|
| **Files Over Databases** | Addresses working state (pending approvals, temp data). This pattern addresses what belongs in structured storage vs narrative. |
| **Per-Agent Data Access Control** | Enforces access control on databases (the persistence layer). Complements this pattern by securing structured data. |
| **Hub-and-Spoke Orchestration** | The hub agent may consolidate memory files and extract facts for the database. Acts as the boundary enforcer. |
| **Multi-Source Memory Architecture** | (Hypothetical pattern) Could describe how agents synthesize from both memory files and database queries. |

---

## Metadata

| Property | Value |
|----------|-------|
| **Contributor** | Sean Erikson, CEO & Co-Founder, Cloud Nirvana |
| **Production Environment** | Multi-agent agentic AI system |
| **First Published** | March 2026 |
| **Last Updated** | March 24, 2026 |
| **Cloud Nirvana Event** | Seed pattern (pre-Q2 2026) |
| **License** | CC BY 4.0 |

---

## Revision History

| Date | Change | Author |
|------|--------|--------|
| 2026-03-24 | Initial publication | Sean Erikson / Lou 🔥 |
