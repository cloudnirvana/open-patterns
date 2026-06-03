# Threshold-Gated Observability for AI Operating Systems

> **One-line intent:** Monitor operational signals across an autonomous AI system, fire alerts only on meaningful state transitions, and surface health status to every dashboard through a single shared contract.

> **⚠️ STATUS: IN PROGRESS** — Pattern is derived from a real incident (runaway token costs in a production AIOS deployment). Architecture is proven; full implementation is in Phase 1 build. Seeking practitioner contributions on signal design and threshold calibration.

## Pattern in 60 Seconds

**The problem:** Autonomous AI systems generate operational signals — token costs, memory pressure, error rates, database health — with no feedback loop. Problems surface only when humans notice them externally, often too late and too expensively. A production AIOS deployment burned $176 in a single day before anyone knew there was a problem.

**The insight:** Treat every operational metric as a named **Signal** with a lifecycle: collect → evaluate → transition → alert. Only alert on *state changes* (healthy→warning, warning→critical), not on every evaluation. Write all signal state to a shared integration contract (ops-cache) that any dashboard can read without coupling.

**The key structure:**
- **Signal registry:** Named signals with collection methods, units, and thresholds — defined in config, not code
- **State-transition alerting:** Alerts fire on transitions, not polling — eliminates alert fatigue
- **Cooldown windows:** Per-severity re-alert intervals — warning every 4h, critical every 1h
- **Shared ops-cache contract:** Single JSON file that decouples signal collection from dashboard display from alert delivery
- **Config-driven thresholds:** YAML config; changing a threshold requires no code deploy

**What broke when we got this wrong:** A misconfigured cron (2-minute interval instead of 15-minute, combined with full workspace context loading on every run) burned 21M+ tokens per day. No alert fired. The first signal was a $176 Anthropic invoice.

---

## Classification

| Property | Value |
|----------|-------|
| **Category** | Production Readiness |
| **Difficulty** | Intermediate |
| **Also Known As** | Operational Health Monitoring, Signal-Based Alerting, Autonomous System Observability |
| **Status** | ⚠️ **IN PROGRESS** — Real incident, architecture proven, implementation in progress |
| **Related Patterns** | REM Cycle (Nightly Maintenance), Escalation Chain with SLA, System Hygiene |

---

## Motivation

You've built an autonomous AI operating system. Agents run on schedules. Crons fire every few minutes. API calls accumulate. Memory grows. Databases drift.

Most of the time, everything is fine. But sometimes:
- A cron interval is misconfigured and fires 10x more often than intended
- Cache retention is set to 5-minute instead of 1-hour, costing 10x more per write
- Gateway RSS climbs to 3.5 GB and starts corrupting tool calls
- A database grows unchecked and starts slowing queries

In all of these cases, the system keeps running. No exceptions are thrown. No services go down. But you're hemorrhaging money, degrading performance, or accumulating technical debt — and you don't know until it's too late.

**The fundamental problem:** Autonomous systems are *designed* to run without human intervention. That's their value. But autonomy without observability is flying blind.

---

## Applicability

Use this pattern when:
- AI agents run autonomously on schedules (crons, heartbeats, background tasks)
- The system makes API calls that generate variable costs
- Multiple operational concerns (memory, cost, error rates, disk) need unified monitoring
- You want alerts that are signal, not noise — state changes, not polling
- You have multiple dashboard surfaces that need the same health data without tight coupling

Do NOT use this pattern when:
- You're building a one-agent system with a single monitored metric (overkill)
- You have an existing observability stack (Datadog, Grafana) already in place — integrate with that instead
- Real-time sub-minute alerting is required (this pattern is 30-minute polling by design)

---

## Structure

```
┌─────────────────────────────────────────────────────────────────┐
│                        SIGNAL REGISTRY                           │
│              signals.yaml — config, not code                     │
│                                                                  │
│  id | label | collector | unit | warning_threshold | critical   │
└────────────────────────┬────────────────────────────────────────┘
                         │ signal definitions
                         ▼
┌────────────────────────────────────────────────────────────────┐
│                        COLLECTORS                               │
│                                                                 │
│  infrastructure.py   cost.py   cron_health.py   database.py   │
│                                                                 │
│  Each collector knows HOW to get one category of signals.      │
│  Collectors are independent — add one without touching others. │
└────────────────────────┬───────────────────────────────────────┘
                         │ current values
                         ▼
┌────────────────────────────────────────────────────────────────┐
│                        EVALUATOR                                │
│                                                                 │
│  value + thresholds → status (healthy / warning / critical)    │
│  current status + previous status → transition detected        │
│  transition + cooldown state → alert decision                  │
└──────────┬─────────────────────────┬──────────────────────────┘
           │ alerts to fire           │ signal states
           ▼                         ▼
┌──────────────────────┐  ┌─────────────────────────────────────┐
│       ALERTER        │  │        OPS-CACHE CONTRACT            │
│                      │  │                                      │
│  Telegram (primary)  │  │  ops-cache.json → meter section     │
│  Webhook (future)    │  │  Read by: Ops console, Mission       │
│  Email (future)      │  │  Control, any future dashboard       │
└──────────────────────┘  └─────────────────────────────────────┘
```

---

## Participants

**Signal Registry (`signals.yaml`):** The canonical list of what to monitor. Defines signal IDs, labels, collection methods, units, thresholds, and alert config. Config, not code — changing thresholds requires no deployment.

**Collectors:** Python modules, one per concern domain (infrastructure, cost, cron health, database, memory). Each collector implements a simple interface: given a signal definition, return a numeric value. Collectors are independent and composable.

**Evaluator:** Compares collected values against thresholds, produces signal status, detects state transitions, and applies cooldown logic. Pure logic — no I/O.

**Alerter:** Delivers alerts to configured channels. Respects cooldown windows. Formats alerts consistently. Channels are pluggable — Telegram is the default; adding email or webhook requires no changes to the evaluator or collectors.

**State Store (`meter-state.json`):** Persists signal states between runs. Required for transition detection (can't know you went from warning → critical without knowing you were warning). Includes last-alerted timestamps for cooldown enforcement.

**Ops-Cache Contract (`ops-cache.json`):** The shared integration point. Meter writes its signal summary here. Dashboards read from here. Neither side knows about the other — only the schema matters.

---

## Collaboration (Sequence)

```
Every 30 minutes:
  1. meter.py starts
  2. Load signals.yaml → N signal definitions
  3. Load meter-state.json → previous states
  4. For each signal:
       collector.collect(signal) → value
       evaluator.evaluate(value, signal.thresholds) → status
       evaluator.detect_transition(status, previous_status) → transition
  5. Write new states → meter-state.json
  6. Write signal summary → ops-cache.json (meter section)
  7. For each signal with a transition:
       if within cooldown: skip
       else: alerter.send(signal, transition, channels)
```

---

## Implementation

### signals.yaml

```yaml
signals:
  - id: cost.daily
    label: "Daily API Spend"
    collector: cost
    method: daily_spend_usd
    unit: USD
    thresholds:
      warning: 15
      critical: 40
    alert:
      channels: [telegram]
      cooldown_warning_minutes: 240
      cooldown_critical_minutes: 60

  - id: gateway.rss
    label: "Gateway RSS Memory"
    collector: infrastructure
    method: gateway_rss_mb
    unit: MB
    thresholds:
      warning: 1500
      critical: 2000
    alert:
      channels: [telegram]
      cooldown_warning_minutes: 240
      cooldown_critical_minutes: 60
```

### Collector Interface

```python
class BaseCollector:
    def collect(self, signal_id: str, method: str) -> float | None:
        """
        Collect the current value for a signal.
        Returns float value, or None if collection fails.
        Failures are non-fatal — signal status becomes 'unknown'.
        """
        raise NotImplementedError
```

### Evaluator Logic

```python
def evaluate(value, thresholds) -> str:
    if value is None:
        return "unknown"
    if value >= thresholds.critical:
        return "critical"
    if value >= thresholds.warning:
        return "warning"
    return "healthy"

def detect_transition(current_status, previous_status) -> str | None:
    if current_status != previous_status:
        return f"{previous_status} → {current_status}"
    return None
```

### Alert Format

```
🔴 METER ALERT — Critical

Signal:   Daily API Spend
Value:    $47.23
Critical: > $40.00
Change:   Warning → Critical

Time: 2026-06-03 14:30 ET
```

### ops-cache.json Schema (meter section)

```json
{
  "meter": {
    "last_run_at_ms": 1780507261756,
    "last_run_iso": "2026-06-03T14:30:00Z",
    "status": "warning",
    "signal_count": 12,
    "healthy_count": 10,
    "warning_count": 2,
    "critical_count": 0,
    "signals": [
      {
        "id": "cost.daily",
        "label": "Daily API Spend",
        "value": 18.42,
        "unit": "USD",
        "status": "warning",
        "threshold_warning": 15,
        "threshold_critical": 40,
        "trend": "down",
        "evaluated_at": "2026-06-03T14:30:00Z"
      }
    ]
  }
}
```

---

## Consequences

### Benefits

**Alert quality over quantity.** State-transition alerting with cooldown windows means every alert you receive represents a genuine change in system health. The system doesn't cry wolf.

**Config-driven extensibility.** Adding a new signal takes 10 lines of YAML. No Python required. Changing thresholds takes 30 seconds. No deployment required.

**Dashboard decoupling.** The ops-cache contract means any dashboard — current or future — gets Meter data automatically. The dashboard doesn't need to know how signals are collected. Meter doesn't need to know what the dashboard looks like.

**Failure isolation.** A collector that fails (e.g., Anthropic API is down) produces an "unknown" status for that signal and moves on. One bad signal doesn't take down the whole health check.

**Incident provenance.** When something goes wrong, you have a timestamped record of every signal state transition. You can reconstruct the timeline of events.

### Tradeoffs

**30-minute polling is not real-time.** A cost spike that starts at 14:00 may not alert until 14:30. For sub-minute alerting requirements, this pattern is insufficient — use Prometheus + Grafana with a proper scrape interval instead.

**Trajectory file analysis is approximate.** Daily cost computed from trajectory files won't exactly match Anthropic's billing (different rounding, timezone differences, sessions not yet flushed to disk). It's directionally correct, not auditable. For billing-grade accuracy, use the provider's usage API directly.

**State file is a single point of failure.** If `meter-state.json` is corrupted or deleted, all transitions look like "first run" and may trigger spurious alerts. Mitigate by keeping the state file small and backing it up with the nightly REM cycle.

---

## Known Uses

**Cloud Nirvana AIOS (2026):** Production deployment monitoring 12 signals across infrastructure, cost, cron health, and database. Originated from a $176 single-day cost incident caused by a misconfigured cron. Implementation in progress as of June 2026.

---

## Related Patterns

**[REM Cycle (Nightly Maintenance)](rem-cycle-nightly-maintenance.md):** Complementary pattern. REM does deep nightly maintenance; Meter does continuous 30-minute signal monitoring. REM is episodic; Meter is continuous.

**[Escalation Chain with SLA](../operations-orchestration/escalation-chain-with-sla.md):** Meter alerts can feed into an escalation chain. A critical signal that stays critical for N hours can auto-escalate through the chain.

**[System Hygiene for Agentic Systems](system-hygiene-for-agentic-systems.md):** System Hygiene defines the practices that keep signals healthy. Meter tells you when hygiene has drifted.

---

## Open Questions (Seeking Practitioner Input)

1. **Threshold calibration:** How do you determine the right warning/critical thresholds for a new signal? What's the process for adjusting them over time based on observed baselines?

2. **Provider API vs. local trajectory analysis:** Have you connected to Anthropic's usage API for real-time cost data? Is it reliable enough for alerting? What's the lag between spend and API visibility?

3. **Per-agent cost signals:** Is per-agent cost granularity worth the signal count multiplication? Or is aggregate daily cost sufficient for most cases?

4. **Multi-channel delivery:** What delivery channels have you used beyond Telegram? Have you wired Meter-style alerting to PagerDuty, OpsGenie, or similar?

5. **Historical trending:** How many readings do you retain before rotating? What's the right window for trend computation (last 3 readings? last 24 hours)?

---

## Practitioner Notes

*This section is for real-world learnings beyond the design. Add yours.*

**2026-06-03, Cloud Nirvana:** The incident that created this pattern. A cron job configured with a 2-minute interval (vs. intended 15 minutes) combined with 5-minute cache retention (vs. 1-hour) generated 160M+ tokens in a single day before anyone noticed. The fix was straightforward once diagnosed; the problem was that no alerting existed to trigger diagnosis. Total cost before discovery: $176. Projected monthly cost if undetected: ~$5,000+. Time to diagnose after manual discovery: ~3 hours. Time to fix: 20 minutes. A $15 daily threshold alert would have fired at ~8 AM and saved the entire day's burn.

---

*This pattern was extracted from the Cloud Nirvana AIOS implementation.*  
*Status: In Progress — real incident, real architecture, implementation underway.*  
*Contributions welcome: open a PR or file an issue at github.com/cloudnirvana/open-patterns*
