# Per-Phase Effort.db with Seeded Prior

> **One-line intent:** A per-phase SQLite database of token-cost observations bootstraps from a cross-project seed file (524 attempts: 261 builder, 263 reviewer; builder median 53,416 tokens; reviewer median 49,499 tokens), then switches to the per-phase median once ≥5 attempts accumulate — so every new phase has a defensible cost estimate from day one, and the estimate improves automatically as the phase builds.

## Pattern in 60 Seconds

_The entire pattern distilled into something anyone can read in under a minute. No jargon, no code. A CEO, an engineer, and an entrepreneur should all understand this section._

**The problem:** Static token-budget thresholds are simultaneously too aggressive for some phases and too permissive for others, and a brand-new project has no historical data at all to work from.

**The insight:** Observations from the current phase — and from sister projects — are better predictors of step cost than any fixed constant, as long as you have a defensible fallback for cold starts.

**The key structure:**

| Phase state | What the system uses |
|-------------|---------------------|
| Phase has fewer than 5 attempts | Seed prior from `effort_baseline.json` (cross-project corpus) |
| Phase has 5 or more attempts | Per-phase median of all attempts with recorded token counts |
| Between phases / new project | Regenerate seed file to incorporate latest observations |

**What broke when we got this wrong:** When applying the pairmode methodology to a new project without a seed prior, the orchestrator had to guess a threshold constant. On one onboarding, the team reused "120k tokens" from the previous project — but the new project used a heavier model. By the midpoint of the first phase, the context had already compacted, dropping in-flight orchestrator state.

---

## Classification

| Property | Value |
|----------|-------|
| **Category** | Cost & Operations |
| **Difficulty** | Intermediate |
| **Also Known As** | Seeded Cost Prior, Bootstrapped Effort Baseline, Cross-Project Token Estimator |

---

## Motivation

_A concrete scenario that illustrates the problem. Tell a short story. Make the reader say "I've been there."_

A team adopts the pairmode methodology for a new project. They set the context budget threshold to 120,000 tokens — "because that's what we used on the last project." The previous project used Sonnet for all sub-agents. This project uses Opus. Opus burns roughly three times the tokens per story. By story 7 of the first phase (12 stories planned), the context window has hit 140,000 tokens. The Claude Code CLI compacts mid-phase, silently dropping the orchestrator's in-flight knowledge of which stories are complete and which are in progress. The team spends two hours reconstructing state from git log.

The root cause is that the threshold was calibrated to the wrong project. The team had no mechanism to import knowledge from previous work, and no feedback loop to improve the estimate as the phase progressed.

The Per-Phase Effort.db with Seeded Prior pattern solves both problems. A seed file — generated from historical observations across one or more projects — provides a defensible starting estimate for a brand-new phase on a brand-new project. As the phase accumulates attempts, the system switches to a per-phase median drawn from actual observations, making the estimate self-improving without any operator intervention.

A secondary problem the pattern addresses is enforcement gap. When the budget check is a manual step the orchestrator is supposed to remember, it gets skipped under load. Encoding the dynamic estimate as a hook-enforced pre-spawn check makes the guard mechanical. CER-027 (2026-05-29) captured this gap precisely: the check was present in documentation but absent in execution.

---

## Applicability

Use this pattern when:
- You are running multi-story phases where context budget overrun causes data loss or compaction
- You are onboarding new projects that should benefit from sister-project token observations
- You want the cost estimate to improve automatically over the life of a phase without manual tuning
- You need the same code path to handle cold-start and warm-phase transparently

Do NOT use this pattern when:
- Phases are single-story (no meaningful per-phase median accumulates)
- The project's token profile is wildly different from the seed corpus (e.g., embedding-only pipelines, batch inference jobs with no prose generation)
- You cannot accept the SQLite disk I/O overhead (negligible in practice, but relevant in extremely I/O-constrained environments)

---

## Structure

_Visual representation of the pattern using Mermaid diagrams._

```mermaid
graph TD
    A[New phase starts] --> B[Load effort.db for current phase]
    B --> C{N attempts with\nnon-null tokens_total?}
    C -- "N < 5" --> D[Use seed prior\nfrom effort_baseline.json]
    C -- "N >= 5" --> E[Compute per-phase median\nof all recorded attempts]
    D --> F[Compute context_budget_threshold\n= threshold × 1 + overrun_pct]
    E --> F
    F --> G{Pre-spawn check:\ncurrent + expected > ceiling?}
    G -- No --> H[Spawn builder or reviewer sub-agent]
    G -- Yes --> I[Emit CONTEXT BUDGET prompt\nWait for operator acknowledgment]
    I --> H
    H --> J[After attempt: record tokens_total,\ntool_uses, duration_ms, outcome,\nmodel, agent_role in effort.db]
    J --> C
    J --> K[Phase end: optionally regenerate\neffort_baseline.json for downstream projects]
```

_The loop shows how each recorded attempt feeds back into the N-count check: once the fifth attempt lands, the system automatically switches from seed prior to per-phase median on the next pre-spawn evaluation._

---

## Participants

| Participant | Role | Example |
|------------|------|---------|
| `effort.db` | Per-phase SQLite database; stores one row per sub-agent attempt with token counts, duration, outcome, model, and agent role | `~/.claude/projects/<hash>/effort.db` scoped to the current project |
| `effort_baseline.json` | Cross-project seed file; provides role-stratified medians and percentiles for cold-start estimation | `skills/pairmode/seed/effort_baseline.json`; 524 attempts (261 builder, 263 reviewer); source project list not captured at generation time |
| `context_budget.py` | Pure-logic module; owns `estimate_next_step_tokens()` and `should_block()`; no side effects on import | `skills/pairmode/scripts/context_budget.py` |
| `pre_tool_use.py` hook | Thin enforcement delegate; reads state.json, calls `context_budget.py`, emits block decision before each Task spawn | `hooks/pre_tool_use.py`; one stdin parse, one tool-name check, one delegate call, one stdout emit |
| `state.json` | Configurable tunables; holds `expected_step_tokens` (seeded prior), `context_budget_threshold` (default 120,000), `context_budget_overrun_pct` (default 10%), `context_budget_reprompt_margin` (default 10,000) | `.companion/state.json` |
| Orchestrator | Caller that spawns builder and reviewer sub-agents; receives the block prompt when ceiling is projected to be exceeded | Claude Code orchestrator session |

---

## How It Works

_Step-by-step description of how the participants collaborate._

1. **Phase start.** The orchestrator begins a new phase. `effort.db` for the current phase may be empty (first story of the phase) or may contain rows from earlier stories in the same phase.

2. **Count check.** Before spawning a sub-agent, `context_budget.py::estimate_next_step_tokens()` queries `effort.db` for all rows where `phase = <current_phase> AND tokens_total IS NOT NULL`. The query applies no outcome filter — all attempts with a recorded token count are included in the median, regardless of whether they passed or failed.

3. **Prior vs. median.** If the row count is fewer than 5, the function returns `seeded_default` — the value of `expected_step_tokens` from `state.json`, which was bootstrapped from `effort_baseline.json` at project initialization. If the row count is 5 or more, the function returns `int(statistics.median(values))` computed from the actual per-phase observations.

4. **Ceiling computation.** `context_budget.py::should_block()` computes the ceiling: `threshold × (1 + overrun_pct)`. With defaults, that is `120,000 × 1.10 = 132,000` tokens.

5. **Pre-spawn gate.** The hook compares `current_tokens + expected_next` against the ceiling. If the sum exceeds the ceiling (and the operator has not already acknowledged at the current token level), `should_block()` returns `True` and the hook emits a CONTEXT BUDGET prompt. The orchestrator waits for operator acknowledgment or a `/clear` before proceeding.

6. **Attempt recording.** After each sub-agent completes, its result (tokens_total, tool_uses, duration_ms, outcome, model, agent_role, phase) is appended to `effort.db`. This immediately affects the next count check.

7. **Phase end / seed refresh.** When the phase is complete, operators may run `refresh_effort_baseline.py` to regenerate `effort_baseline.json` from all projects in scope. The refreshed seed is then available to any new project that initializes from the same repository.

### Code / Configuration Example

```python
# skills/pairmode/scripts/context_budget.py (abridged)

def estimate_next_step_tokens(
    db_path: Path | None,
    phase: str | None,
    seeded_default: int,
) -> int:
    """Return the per-phase median if >= 5 attempts are recorded;
    otherwise return seeded_default.

    NOTE: No outcome filter is applied. All attempts with a non-null
    tokens_total are included in the median, regardless of pass/fail.
    """
    if db_path is None or phase is None:
        return seeded_default
    # ... (db open / error handling elided) ...
    cur.execute(
        "SELECT tokens_total FROM attempts "
        "WHERE phase = ? AND tokens_total IS NOT NULL",
        (phase,),
    )
    rows = cur.fetchall()
    values = [int(r[0]) for r in rows if r and r[0] is not None]
    if len(values) < 5:   # N=5 switch threshold
        return seeded_default
    return int(statistics.median(values))
```

```json
// skills/pairmode/seed/effort_baseline.json
{
  "generated_at": "2026-05-29T00:00:00Z",
  "source_projects": [],
  "by_role": {
    "builder": {"n": 261, "median": 53416, "p75": 77498, "p90": 111434},
    "reviewer": {"n": 263, "median": 49499, "p75": 58308, "p90": 75349}
  }
}
```

The seed file carries role-stratified statistics. `source_projects` is an empty array — the project source list was not captured at generation time. The builder median (53,416 tokens) and reviewer median (49,499 tokens) serve as `expected_step_tokens` defaults in `state.json` until ≥5 per-phase observations accumulate.

---

## Consequences

### Benefits
- New projects get a defensible cost estimate from day one instead of an arbitrary constant
- The estimate improves automatically as the phase builds — no operator tuning required
- The same code path handles cold start and warm phase transparently; no branching in the orchestrator
- Cost anomalies (e.g., a step consuming 3× the phase median) surface automatically via the pre-spawn gate
- The seed corpus grows over time as more phases complete and refresh the baseline

### Liabilities
- The seed corpus must be actively maintained as projects diverge in token profile; a stale corpus degrades estimate quality
- The per-phase median lags reality during the first four attempts of every phase — cold-start accuracy depends entirely on seed quality
- SQLite adds disk I/O at every pre-spawn check (negligible in practice but non-zero)
- The N=5 threshold is a hard-coded constant; projects with very short phases (2–4 stories) will never leave seed-prior mode

### What Broke in Practice

- **Pre-INFRA-127 (all phases before Phase 47):** The context budget threshold was a static constant (120,000 tokens). Sessions running small stories with fast models hit the threshold at a very different real-cost point than sessions running large stories with expensive models. The static threshold was simultaneously too aggressive for high-cost sessions (blocking too early) and too permissive for low-cost ones (missing the warning entirely).

- **New-project cold-start problem:** When pairmode was applied to a new project (e.g., cora, aab), the per-phase median had no data. The orchestrator had to guess or reuse a constant from the previous project. In one case this meant reusing a threshold calibrated to Sonnet on a project that ran Opus, resulting in mid-phase context compaction and lost orchestrator state.

- **CER-027 enforcement gap (2026-05-29):** The context budget check was documented but not mechanically enforced. A static constant in a prompt is not self-enforcing. CER-027 captured this: the check existed as a step the orchestrator was supposed to remember, but under load it was skipped. The per-phase median combined with hook enforcement made the estimate dynamic and the check mechanical — neither depends on the orchestrator's attention.

---

## Implementation Notes

_Practical guidance for teams adopting this pattern._

### Variations

- **Role-stratified seed prior:** The seed file carries separate builder and reviewer medians. `state.json` can store separate `expected_step_tokens` values per role if the project's builder/reviewer cost ratio differs significantly from the corpus.
- **Percentile-based threshold:** Instead of using the median as the per-step estimate, teams with high variance across stories may prefer the p75 value from the seed file (77,498 tokens for builder, 58,308 for reviewer) as a more conservative starting point.
- **Phase-to-phase carry-forward:** Rather than resetting the seed at project initialization only, some teams regenerate `effort_baseline.json` at the end of every phase, so each phase starts with an estimate that includes the previous phase's observations.

### Common Pitfalls

- **Assuming outcome filtering:** The per-phase median includes all attempts with a recorded `tokens_total`, regardless of pass/fail outcome. Failed attempts are not excluded. Do not assume the median reflects only successful stories — it reflects all observed step costs.
- **Treating N=5 as tunable without testing:** The switch threshold is a constant in code, not a config value. Changing it requires modifying `context_budget.py` and running the test suite.
- **Forgetting to initialize state.json:** On project initialization, `expected_step_tokens` must be set from the seed file. If it remains at the default constant, the cold-start benefit is lost.
- **Letting the seed corpus go stale:** If `refresh_effort_baseline.py` is never run after the initial seed, the corpus remains frozen at generation time. Significant project evolution (new models, new story sizes) will degrade estimate quality over time.

---

## Security Implications

_Every pattern has security considerations. This section is mandatory._

### Attack Surface

- Both `effort.db` and `effort_baseline.json` are local files with no network exposure. The pattern introduces no new network endpoints or remote calls.
- The file paths for `effort.db` and `effort_baseline.json` are configurable via `state.json`. An attacker who can write to `state.json` can redirect the system to read from an arbitrary SQLite file or JSON file. The trust boundary is the same as other `state.json` fields — user-owned, local-only, no privilege escalation beyond what the Claude Code CLI already has.

### Data Sensitivity

- `effort.db` contains token counts, tool-use counts, duration in milliseconds, outcome labels, model names, and agent role labels. It contains no user content, no secrets, no API keys, and no personally identifiable information.
- `effort_baseline.json` contains aggregate statistics (counts, medians, percentiles) stratified by role. No individual attempt data is present in the seed file. No project names are stored in the current generation (`source_projects: []`).

### Failure Modes

- If `effort.db` is corrupted or unreadable, `estimate_next_step_tokens()` returns `seeded_default` (graceful degradation to the seed prior). The hook does not crash; the orchestrator continues with a potentially less accurate estimate.
- If `effort_baseline.json` is missing or malformed, `state.json` must already contain a valid `expected_step_tokens` value. If both are absent, the threshold math falls back to the hard-coded constant in `context_budget.py`. No data loss occurs, but the cold-start estimate becomes a guess.
- A malicious `effort_baseline.json` with extreme median values could cause the pre-spawn gate to block aggressively (very low values) or never fire (very high values). Impact is limited to orchestrator workflow disruption; no code execution or data exfiltration is possible through this vector.

### Mitigations

- Treat `state.json`, `effort.db`, and `effort_baseline.json` with the same access controls as other local Claude Code project state. On shared development machines, ensure project directories have appropriate permissions.
- The `refresh_effort_baseline.py` CLI should be run by the project owner, not automated through a writable hook. Automating seed regeneration from an untrusted source is not recommended.
- Validate `effort_baseline.json` schema before use (the current loader accepts any JSON; a schema check would harden against crafted files).

---

## Known Uses

| Organization | Context | Scale |
|-------------|---------|-------|
| flex project (cloudnirvana) | effort.db used across Phases 22–47; effort_baseline.json seeded from 524 attempts (261 builder, 263 reviewer); source project list not captured at generation time; pattern active in production on 5 downstream projects: forqsite, radar, asp, aab, cora | Team |

---

## Related Patterns

| Pattern | Relationship |
|---------|-------------|
| `context-cost-control` | Budget enforcement strategy; this pattern is the estimation primitive that cost-control patterns build on |
| `context-lifecycle-management` | Decides when to checkpoint based on context pressure; feeds from this pattern's threshold estimate |
| `cer-backlog-living-phases` (NP-3) | CER-027 was the finding that forced mechanical enforcement of the budget check; the two patterns are historically linked — CER-027 identified the enforcement gap that made the dynamic-median hook necessary |

---

## Metadata

| Property | Value |
|----------|-------|
| **Contributor** | David Hague, flex project (david@halfhorse.com) |
| **Production Environment** | macOS/Linux, Python/uv, SQLite, Claude Code CLI |
| **First Published** | 2026-05-31 |
| **Last Updated** | 2026-05-31 |
| **Cloud Nirvana Event** | — |
| **License** | CC BY 4.0 |

---

## Revision History

| Date | Change | Author |
|------|--------|--------|
| 2026-05-31 | Initial publication | David Hague |
