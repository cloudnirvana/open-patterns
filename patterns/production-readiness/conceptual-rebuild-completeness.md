# Conceptual Rebuild Completeness

> **One-line intent:** Every new database table introduced by a phase must answer "where does a human manage this data?" before the phase is checkpointed — ensuring that schema changes always have an operator surface, not just a migration.

## Pattern in 60 Seconds

_The entire pattern distilled into something anyone can read in under a minute. No jargon, no code. A CEO, an engineer, and an entrepreneur should all understand this section._

**The problem:** Phases ship schema changes — new database tables created by migrations — that no human can inspect without a database console. The user interface (UI) to manage that data is always "coming next phase," and next phase never comes.

**The insight:** Schema changes are features; a feature is incomplete until it has an operator surface. The right moment to enforce this is at the gate before building, not in a retrospective after three months of orphaned data.

**The key structure:**

| Situation | Gate outcome |
|-----------|-------------|
| Phase introduces a new table AND a management surface story exists in the phase | Proceed normally |
| Phase introduces a new table AND the story spec documents an accepted exception | Proceed normally (exception audited in spec) |
| Phase introduces a new table AND no management surface AND no exception | PRE-STORY BLOCK — add a UI story or document the exception before building |

**What broke when we got this wrong:** A team introduced three new tables in Phase N, planning admin views for Phase N+2. Phase N+2 never shipped — other priorities intervened. Operators began running raw SQL queries for routine data management. The tables accumulated data visible only to a database administrator (DBA). With this pattern, that decision would have been forced at schema introduction time.

---

## Classification

| Property | Value |
|----------|-------|
| **Category** | Production Readiness |
| **Difficulty** | Intermediate |
| **Also Known As** | Schema-to-UI Parity Rule, Management Surface Gate, UI Completeness Gate |

---

## Motivation

Imagine an AI (artificial intelligence) agent introduces a new `recommendations` table to persist cross-session suggestions as part of Phase 12 of an ongoing project. The phase ships. The service layer passes tests, the migration runs cleanly, and the checkpoint is tagged.

Three months later, an operations analyst asks: "How many recommendations have been generated this week?" The answer requires a database console query. There is no admin view, no API (application programming interface) endpoint, no dashboard widget. The on-call developer writes a one-off SQL (Structured Query Language) query at 2am. The next week, the analyst asks again — same answer, different query, another 2am.

The root cause is not negligence. The team intended to build the management UI. It just never made it into a phase spec. "We'll do it later" is a perfectly rational response to schedule pressure — and it is precisely the wrong one when applied to a schema change that produces data from day one.

The Conceptual Rebuild Completeness pattern prevents this by making the management surface a phase requirement rather than a backlog aspiration. The pattern is enforced by the pre-story schema gate in `CLAUDE.build.md`: before any story that introduces a new schema object is built, the orchestrator checks whether a management surface story exists in the same phase. If it does not, and if no documented exception applies, the gate blocks the build.

A secondary failure mode the pattern addresses is the exception-without-acknowledgment problem. Before the rule was formalized, tables that genuinely did not need a management UI — append-only audit logs, for instance — were sometimes introduced without any notation. Phase reviewers could not distinguish "the UI story is missing by oversight" from "the UI story is missing because the table is append-only and the team decided that consciously." The pattern requires explicit exception acknowledgment in the story spec. Absence of a UI story is acceptable only when its absence is deliberate and documented.

---

## Applicability

Use this pattern when:
- A phase introduces any new persistent database table, collection, or index
- The system will have ongoing production data that operators need to inspect or manage
- You want to enforce UI completeness before phase checkpoint (tagged release), not after
- An agentic build loop drives phase execution and can run the gate mechanically before spawning a builder sub-agent

Do NOT use this pattern when:
- The new schema object qualifies for one of the three documented exceptions (append-only audit/event log, pure junction table, or cron-output cache — see How It Works)
- The project has no UI or operator surface at all (pure API or command-line interface (CLI) project — the equivalent gate is "does the CLI expose management commands for this data?")
- A phase contains only a single hotfix story with no schema changes

---

## Structure

_Decision tree governing the pre-story schema gate. The orchestrator runs this check once per story, before spawning the builder._

```mermaid
graph TD
    A[Story spec read by orchestrator] --> B{Does this story introduce\na new persistent schema object\n(table, collection, index, migration)?}
    B -- No --> C[Skip gate — proceed to builder]
    B -- Yes --> D{Scan remaining stories in phase:\ndoes any story provide a\nmanagement surface for this object?}
    D -- Yes --> C
    D -- No --> E{Does the story spec contain\nan explicit documented exception?\n(append-only / junction table /\ncron-output cache)}
    E -- Yes: exception documented --> F[Proceed — exception is audited in spec]
    E -- No exception documented --> G[PRE-STORY BLOCK\nReport missing management surface\nRequire operator to add UI story\nor document exception before building]
```

_An exception is not a loophole — it is an audited decision. The spec must name which exception applies and explain why. A reviewer can inspect the spec and immediately verify that the absence of a UI story was deliberate._

---

## Participants

| Participant | Role | Example |
|------------|------|---------|
| **Story spec** | The detailed spec for a single story (phase (a scoped unit of work), story, acceptance criteria, primary files). The schema gate reads this to determine whether a new schema object is introduced and whether an exception is documented. | `docs/stories/INFRA/INFRA-127.md` |
| **Phase doc** | The living manifest listing all stories in a phase with their states (`planned`, `in_progress`, `complete`, `deferred`). The gate scans remaining stories in the phase doc to find any management surface story. | `docs/phases/phase-47.md` |
| **Pre-story schema gate** | The mechanical enforcement layer in `CLAUDE.build.md` (lines 134–176). Runs once per story before the builder sub-agent is spawned. The orchestrator follows the gate logic and blocks or proceeds based on the result. | `CLAUDE.build.md` — "Pre-story schema gate" section |
| **Management surface story** | A story within the same phase that provides a route, page, CLI command, or UI component where a human can read, create, update, or delete the data without a database console. CRUD (Create, Read, Update, Delete) is the standard. | An `AdminRecommendationsTable` page story in the same phase as the `recommendations` table migration |
| **Exception acknowledgment** | An explicit note in the story spec naming which of the three accepted exception categories applies and justifying the absence of a UI story. Absence without acknowledgment is not acceptable. | A spec note: "Exception: append-only audit log. No human-editable fields. Observable via existing audit route." |
| **Checkpoint gate** | The final gate before a phase is tagged as complete. Verifies that every planned story is either `complete` or formally deferred, and that no schema-without-UI slipped through without an acknowledged exception. Enforced by the checkpoint sequence in `CLAUDE.build.md`. | `cp47-pairmode-methodology-consolidation` — the Phase 47 checkpoint tag |
| **Orchestrator** | The agent or human who drives story state transitions, runs the pre-story schema gate, and blocks or proceeds. The orchestrator owns the enforcement decision; the gate logic is the authority it follows. | Claude Code orchestrator session running Phase 47 |

---

## How It Works

The following rule is the canonical authority for this pattern. It lives in the global `CLAUDE.md` under `## Conceptual rebuild completeness` and applies to every project and every session:

---

> When a phase introduces a new system — a new data model, an enforcement layer, a
> processing pipeline, or any mechanism that produces or manages persistent state — that
> phase is **not complete** when the service layer passes tests. It is complete only when
> the system is observable and manageable from the UI without a database console.
>
> **The rule:** Every new database table must have an answer to "where does a human
> manage this data?" before the phase is checkpointed. If no existing route covers it,
> a management UI story must be in the same phase spec — not the backlog.
>
> **Exceptions that do not require a dedicated UI (but must be explicitly noted in the spec):**
> - Append-only audit/event log tables (e.g. `assistant_events`) — observable via logs or a future log viewer
> - Pure junction tables whose both parent tables already have full management UIs
> - Cron-output cache tables that are regenerated on a schedule and have no human-editable fields
>
> If a table qualifies for an exception, the spec must name the exception and explain why.
> Silently omitting the UI story is not acceptable — the absence must be deliberate and documented.
>
> **On scope interpretation:** "Build X with full CRUD, evolve later" means build the full
> CRUD now. "Evolve later" describes the schema evolution path, not permission to skip the CRUD.
> When spec language is ambiguous, resolve toward the broader interpretation, not the narrower one.

---

The mechanical enforcement of this rule lives in `CLAUDE.build.md` under the "Pre-story schema gate" section (lines 134–176). That gate is the operational implementation: the orchestrator runs it once per story before spawning the builder sub-agent. It encodes the three accepted exception categories verbatim, and its block message (`PRE-STORY BLOCK`) names the schema object and the two resolution options (add a UI story or document an exception). The gate is not a reminder — it is a hard stop.

Operationally, the orchestrator follows these numbered steps:

1. **Read the story spec.** Determine whether the story introduces a new persistent schema object (table, collection, index, or migration that creates or alters durable storage).
2. **If no schema change:** skip the gate and proceed to Step 1 of the build loop (spawn the builder).
3. **If schema change detected:** scan all remaining stories in the phase doc. Look for any story that provides a management surface — a route, page, command, or component where a human can read, create, update, or delete the data without a database console.
4. **If management surface found:** proceed normally.
5. **If no management surface:** check the current story's spec for an explicit exception note. The three accepted exception categories are:
   - **Append-only audit/event log tables** (e.g., `assistant_events`): immutable event records, observable via logs or a future log viewer. No human-editable fields.
   - **Pure junction tables:** both parent entities already have full management UIs. The junction table itself requires no additional UI because it is managed implicitly through the parents.
   - **Cron-output cache tables:** rows are regenerated on a schedule and have no human-editable fields. Operators have no need to modify the data directly.
6. **If exception is documented in the spec:** proceed normally. The exception is audited in the spec and will be reviewed at checkpoint.
7. **If no management surface and no documented exception:** BLOCK. Emit the PRE-STORY BLOCK report. Do not spawn the builder until the operator has either added a management surface story to the phase spec or documented an exception in the story spec.
8. **Checkpoint gate:** before tagging the phase, verify that every schema-introducing story either has a corresponding management surface story marked `complete`, or has a documented exception in its spec. No schema-without-UI may silently survive to a checkpoint tag.

### Code / Configuration Example

The pre-story schema gate block message, as defined in `CLAUDE.build.md`:

```
PRE-STORY BLOCK — Story [RAIL-NNN] introduces schema object `<name>` with no
management surface in this phase.

A persistent schema change without an administrative surface is an incomplete feature.
Options:
1. Add a management UI story to the phase spec before building.
2. Note an explicit exception in the story spec (append-only, junction table, or
   cron-output cache) if one of those categories applies.
```

A story spec exception acknowledgment (the spec author's responsibility):

```markdown
## Schema exception

This story introduces the `assistant_events` table.

Exception: append-only audit/event log table. Rows are written by the assistant
pipeline and are never edited or deleted by operators. The table is observable via
the existing audit log route and will be surfaced in a future log viewer. No
management UI story is required for this phase.
```

A management surface story entry in the phase Stories table:

```markdown
| INFRA-131 | INFRA | Admin: recommendations table management view | planned |
```

This story satisfies the gate for any story in the same phase that introduces the `recommendations` table migration.

---

## Consequences

### Benefits
- Every schema change ships with an operator surface; "we'll do it later" is blocked at the gate before any code is written.
- Exception acknowledgments are audit-trailed in the story spec — a reviewer can verify in seconds whether the absence of a UI story was deliberate.
- Phase completeness is verifiable without reading every migration file; the gate enforces the invariant at build time.
- The scope interpretation rule ("CRUD now, evolve later" means CRUD now) prevents spec ambiguity from becoming a loophole.

### Liabilities
- Adds one check per story for every phase with schema changes; this is low overhead for most stories but non-zero.
- May feel onerous for small lookup or config tables that have obvious management paths — the gate still fires and requires either a UI story or an explicit exception note.
- Exception creep: if exceptions are granted too liberally or without genuine justification, the gate loses meaning. The "explicitly noted in spec" requirement is the only safeguard against this.

### What Broke in Practice

- **"UI later" decay pattern:** The canonical failure mode is a team that introduces new tables in Phase N, intending to build admin views in Phase N+2. Phase N+2 never comes — other priorities intervene — and the tables accumulate data that only a DBA can inspect. Operators start running raw SQL queries for routine data management. The pattern forces this decision at schema introduction time, not "later." Once the decision is deferred into a backlog, the probability of it shipping falls sharply.

- **Exception-without-acknowledgment:** Before the rule was formalized, append-only audit log tables (such as `assistant_events`) were sometimes introduced without explicitly noting the exception. This left phase reviewers unable to distinguish "the UI story is missing by oversight" from "the UI story is missing because the table is append-only." Both looked identical from the outside: a schema change with no accompanying UI story. The pattern requires explicit exception acknowledgment — absence without documentation is not acceptable, regardless of how obvious the exception may seem to the author.

---

## Implementation Notes

### Variations

- **CLI-only projects:** If the project has no UI, the equivalent gate is "does the CLI expose management commands for this data?" A new table without a corresponding CLI subcommand (e.g., `flex recommendations list`, `flex recommendations delete`) fails the same gate. The exception categories remain identical.
- **API-only projects:** A REST (Representational State Transfer) or GraphQL API endpoint that exposes CRUD operations for the new table satisfies the gate, even if no human-facing UI exists. The key criterion is "without a database console" — an API endpoint qualifies.
- **Deferred management surface:** If the management UI story is in the phase but deferred before the phase is checkpointed, the gate is not satisfied. A deferred story does not count as a management surface. The UI story must be `complete` at checkpoint or the table must have a documented exception.

### Common Pitfalls

- **Treating the exception as informal:** The exception must appear in the story spec, not in a conversation or a comment. If the spec does not contain the exception note, the gate will block the next session that reads it cold.
- **Forgetting scope interpretation:** "We'll build full CRUD later" is not an exception. "Evolve later" describes schema evolution, not permission to defer the management surface. Read the scope interpretation clause in the canonical rule (quoted above) before claiming a deferral.
- **Conflating junction tables with association tables:** A junction table qualifies for the exception only if both parent entities already have full management UIs. A junction table between a new table (with no UI yet) and an existing table does not qualify — the new table still lacks a management surface.
- **Checkpoint tag applied before gate verification:** The checkpoint gate is the last enforcement point. If the tag is applied manually without running the checkpoint sequence, a schema-without-UI can survive into the historical record. The `CLAUDE.build.md` checkpoint sequence includes this as a mandatory verification step.

---

## Security Implications

### Attack Surface

- The pre-story schema gate is a process check operating on local Markdown files. It introduces no new network endpoints, no new code execution paths, and no new attack surface.
- Exception acknowledgments live in story spec files in version control. An attacker who can write to the spec files could forge an exception note, bypassing the gate. The trust boundary is the same as the rest of the codebase — if you can write to the repo, you can modify the specs.

### Data Sensitivity

- The gate logic reads story spec files and phase doc files. These are internal project records. They contain architectural decisions, story descriptions, and exception justifications — all of which should be treated with the same access controls as the rest of the codebase.
- No credentials, tokens, or user data should appear in a story spec or exception note. If an exception acknowledgment touches sensitive table names, it should describe the table's purpose at an appropriate level of abstraction.

### Failure Modes

- **Gate bypassed via exception abuse:** If exception categories are applied without genuine justification — for example, calling a mutable config table an "audit log" — the gate passes and a management surface is never built. The mitigation is review: exception notes in story specs are readable by any cold-eyes reviewer or future orchestrator. Unjustified exceptions are detectable and correctable at review time.
- **Gate not run:** If the orchestrator skips the pre-story schema gate (e.g., because it is operating from a cached plan), schema-without-UI stories can be built without a block. The checkpoint gate provides a second enforcement point, but it fires later. The mitigation is to keep the gate logic in `CLAUDE.build.md` as an explicit, named step that precedes every builder spawn — not an informal reminder.
- **Management surface story deferred before checkpoint:** A UI story that is planned but deferred before the phase tags does not satisfy the gate. If the checkpoint sequence does not verify the `complete` status of management surface stories, the invariant breaks silently. The checkpoint sequence in `CLAUDE.build.md` includes this verification as a mandatory step.

### Mitigations

- Keep the gate logic in `CLAUDE.build.md` as a named, explicit section that precedes every builder spawn. A gate that exists only in memory or as a prompt suggestion is not enforced.
- At each cold-eyes review, scan all story specs for exception notes. Verify that each exception falls into one of the three accepted categories and that the justification is genuine.
- At checkpoint, verify that every schema-introducing story either has a `complete` management surface story or has a documented exception. Do not tag until this check passes.

---

## Known Uses

| Organization | Context | Scale |
|-------------|---------|-------|
| flex project — 5 downstream projects (forqsite, radar, asp, aab, cora) | Enforced since Phase 40 via the Pre-story schema gate in `CLAUDE.build.md`. Global policy encoded in `~/.claude/CLAUDE.md` under "Conceptual rebuild completeness." The gate runs before every builder sub-agent spawn for any story that introduces a new schema object. | Team |

---

## Related Patterns

| Pattern | Relationship |
|---------|-------------|
| `system-hygiene-for-agentic-systems` | Both gate production-readiness. This pattern is the schema parity specialization: where system-hygiene covers general agentic hygiene, Conceptual Rebuild Completeness focuses specifically on the table-to-UI invariant. |
| `phase-spec-pause-resume` (NP-2) | The checkpoint gate (Step 5 in that pattern) also verifies this pattern's compliance before tagging. A phase cannot be checkpointed with a schema-without-UI unless an exception is documented. The two gates fire in sequence at checkpoint time. |
| `builder-reviewer-sub-agent-loop` (NP-1) | The orchestrator runs the pre-story schema gate before spawning the builder sub-agent — it is a pre-build check, not a review-time check. The builder/reviewer loop operates within phases governed by this gate. |

---

## Metadata

| Property | Value |
|----------|-------|
| **Contributor** | David Hague, flex project (david@halfhorse.com) |
| **Production Environment** | Local agentic development, Claude Code plugin, Python/Anthropic SDK |
| **First Published** | 2026-05-31 |
| **Last Updated** | 2026-05-31 |
| **Cloud Nirvana Event** | — |
| **License** | CC BY 4.0 |

---

## Revision History

| Date | Change | Author |
|------|--------|--------|
| 2026-05-31 | Initial publication | David Hague |
