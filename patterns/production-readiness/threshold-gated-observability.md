# Threshold-Gated Observability for AI Operating Systems

> **One-line intent:** Monitor operational signals across an autonomous AI system, fire alerts only on meaningful state transitions, and surface health status to every dashboard through a single shared contract.

> **STATUS: IN PROGRESS** — Pattern is fully implemented in production. Architecture proven, first real alerts caught and resolved. Seeking practitioner contributions on signal design, threshold calibration, and multi-agent cost attribution.

## Pattern in 60 Seconds

**The problem:** Autonomous AI systems generate operational signals — token costs, memory pressure, error rates, database health — with no feedback loop. Problems surface only when humans notice them externally, often too late and too expensively.

**The incident that created this pattern:** A misconfigured cron job fired every 2 minutes instead of 15. Combined with 5-minute cache retention (the default), each run rewrote ~73K tokens at $3.75/MTok. At 720 runs/day: 52M tokens/day. No alert fired. First signal: a $176 Anthropic invoice. Time to diagnose: 3 hours. Time to fix once found: 20 minutes. A $15 daily threshold alert would have fired at 8 AM.

**The insight:** Treat every operational metric as a named **Signal** with a lifecycle: collect → evaluate → transition → alert. Only alert on *state changes* (healthy→warning, warning→critical), not on every evaluation. Write all signal state to a shared integration contract (ops-cache) that any dashboard can read without coupling.

**The five key elements:**
1. **Signal registry (config, not code):** Named signals in YAML — add a new signal in 15 lines, change a threshold in 30 seconds, no deploy required
2. **State-transition alerting:** Alerts fire on changes, not polls — eliminates alert fatigue completely
3. **Cooldown windows:** Per-severity re-alert intervals (warning: 4h, critical: 1h) — recovery always immediate
4. **Shared ops-cache contract:** Single JSON file decouples collection from display from delivery — any dashboard gets Meter data automatically
5. **Collector isolation:** One failed collector = one unknown signal, not a system failure

**What happened when we got this wrong:** The system ran for months with no operational observability. A $176 single-day cost anomaly went undetected until manual discovery.

**What happened after we implemented it:** The first alert fired 30 minutes after deployment. It caught a real bug — the QMD memory index watcher loop silently dying after a gateway restart. Not a false positive. The alert triggered diagnosis, the bug was fixed, and a self-heal was added to the nightly maintenance cycle.

---

## Classification

| Property | Value |
|----------|-------|
| **Category** | Production Readiness |
| **Difficulty** | Intermediate |
| **Also Known As** | Operational Health Monitoring, Signal-Based Alerting, Autonomous System Observability |
| **Status** | IN PROGRESS — Implemented in production, seeking real-world calibration data |
| **Related Patterns** | REM Cycle (Nightly Maintenance), Escalation Chain with SLA, System Hygiene |

---

## Motivation

You've built an autonomous AI operating system. Agents run on schedules. Crons fire every few minutes. API calls accumulate. Memory grows. Databases drift.

Most of the time, everything is fine. But sometimes:
- A cron interval is misconfigured and fires 10x more often than intended
- Cache retention is set to 5-minute instead of 1-hour, costing 10x more per write
- The memory index watcher loop silently dies after a gateway restart
- Gateway RSS climbs to 3.5 GB and starts corrupting tool calls
- A database grows unchecked and starts slowing queries

In all these cases, the system keeps running. No exceptions are thrown. No services go down. But you're hemorrhaging money, degrading performance, or accumulating technical debt — and you won't know until it's too late.

**The fundamental problem:** Autonomous systems are designed to run without human intervention. That's their value. But autonomy without observability is flying blind.

---

## Applicability

Use this pattern when:
- AI agents run autonomously on schedules (crons, heartbeats, background tasks)
- The system makes API calls that generate variable costs
- Multiple operational concerns (memory, cost, error rates, disk) need unified monitoring
- You want alerts that are signal, not noise — state changes, not constant polling
- You have or plan multiple dashboard surfaces that need the same health data

Do NOT use this pattern when:
- You're running a single agent with one monitored metric (overkill)
- You already have a mature observability stack (Datadog, Grafana, PagerDuty) — integrate with that instead
- Sub-minute real-time alerting is required — this pattern is 30-minute polling by design

---

## Structure

```
┌─────────────────────────────────────────────────────────────────┐
│                      SIGNAL REGISTRY                             │
│                   signals.yaml (config)                          │
│                                                                  │
│   id | label | collector | unit | thresholds | alert_config     │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────────────────┐
│                       COLLECTORS                              │
│                                                               │
│  infrastructure  ·  cost  ·  cron_health  ·  database  ·  memory  │
│                                                               │
│  Each module: collect(signal_id, method, **kwargs) → float   │
│  Failure returns None (non-fatal, signal becomes unknown)    │
└────────────────────────┬─────────────────────────────────────┘
                         │ current values
                         ▼
┌──────────────────────────────────────────────────────────────┐
│                       EVALUATOR                               │
│                                                               │
│  value + thresholds + direction → status                     │
│  current status + previous status → transition detected      │
│  transition + cooldown timestamps → alert decision           │
│  current + history → trend (up/down/flat)                    │
└──────────┬──────────────────────────┬────────────────────────┘
           │ alerts                    │ signal states
           ▼                          ▼
┌──────────────────────┐  ┌──────────────────────────────────────┐
│       ALERTER        │  │        OPS-CACHE CONTRACT             │
│                      │  │                                       │
│  Telegram (primary)  │  │  ops-cache.json → "meter" section    │
│  Webhook (future)    │  │  Read by: any dashboard, zero coupling│
│  Email (future)      │  │                                       │
└──────────────────────┘  └──────────────────────────────────────┘
```

---

## Participants

**Signal Registry (`signals.yaml`):** Canonical definition of what to monitor. Config, not code. Owner of thresholds, units, and alert delivery policy per signal.

**Collectors:** Domain-specific Python modules (one per concern). Implement `collect(signal_id, method, **kwargs) → Optional[float]`. Independent and composable — adding one doesn't touch others.

**Evaluator:** Pure logic module. No I/O. Compares values to thresholds, detects transitions, computes trends, applies cooldown logic.

**Alerter:** Delivery module. Channels are pluggable. Currently Telegram via platform CLI. Respects cooldown windows. Formats messages consistently.

**State Store (`meter-state.json`):** Persists signal state between runs. Required for transition detection. Stores last-alerted timestamps for cooldown enforcement. Keep small — rotate if it grows.

**Ops-Cache Contract (`ops-cache.json`):** The shared integration point. Meter writes here. Dashboards read here. Schema-only coupling. Any new dashboard surface gets Meter data automatically.

---

## Collaboration (Execution Sequence)

```
Every 30 minutes (cron, isolated session, lightContext):

1. Load signals.yaml → signal definitions
2. Load meter-state.json → previous states + alert timestamps
3. For each signal:
   a. collectors.collect(signal) → value (or None on failure)
   b. evaluator.build_signal_state(value, signal, previous, history) → SignalState
   c. evaluator.should_alert(state, signal) → bool
      - Only True when: status changed AND not within cooldown
      - Recovery always alerts (no cooldown)
   d. If should_alert: alerter.deliver(state, signal)
   e. Update state: status, value, history, last_alerted timestamps
4. Write new states → meter-state.json
5. Build meter section → write to ops-cache.json
```

---

## Implementation

### signals.yaml Structure

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
      direction: over       # alert when value > threshold
    alert:
      channels: [telegram]
      cooldown_warning_minutes: 240
      cooldown_critical_minutes: 60
    dashboard:
      display: true
      show_bar: true
      bar_max: 60           # scale for fill bar

  - id: cost.cache_efficiency
    label: "Cache Read Ratio"
    collector: cost
    method: cache_read_ratio_pct
    unit: "%"
    thresholds:
      warning: 70
      critical: 50
      direction: under      # alert when value < threshold (lower = worse)
    alert:
      channels: [telegram]
      cooldown_warning_minutes: 240
      cooldown_critical_minutes: 120
    dashboard:
      display: true
      show_bar: true
      bar_max: 100

  - id: gateway.uptime
    label: "Gateway Uptime"
    collector: infrastructure
    method: gateway_uptime_hours
    unit: hours
    thresholds:
      warning: null
      critical: null
      direction: over
    alert:
      channels: []          # empty = no alerts, display only
    dashboard:
      display: true
      informational: true   # shown but never triggers alerts
```

### Collector Interface

```python
# Each collector module implements this function
def collect(signal_id: str, method: str, **kwargs) -> Optional[float]:
    """
    Collect current value for a signal.
    Returns float, or None if collection fails.
    None → signal becomes 'unknown' status (non-fatal).
    kwargs includes: agent=<agent_id> for per-agent signals.
    """
    fn = globals().get(method)
    if not fn:
        return None
    try:
        return fn(**kwargs)
    except Exception as e:
        print(f"[meter/{signal_id}] collection failed: {e}")
        return None
```

### Evaluator Core Logic

```python
def evaluate_status(value: Optional[float], signal: dict) -> str:
    if value is None:
        return "unknown"
    thresholds = signal.get("thresholds", {})
    warn = thresholds.get("warning")
    crit = thresholds.get("critical")
    direction = thresholds.get("direction", "over")

    if direction == "over":
        if crit is not None and value >= crit: return "critical"
        if warn is not None and value >= warn: return "warning"
        return "healthy"
    elif direction == "under":
        if crit is not None and value <= crit: return "critical"
        if warn is not None and value <= warn: return "warning"
        return "healthy"

def should_alert(state: SignalState, signal: dict) -> bool:
    if not state.changed:
        return False                          # same status = no alert
    if state.status == "healthy":
        return True                           # recovery always fires
    # Check cooldown
    alert_cfg = signal.get("alert", {})
    if state.status == "warning":
        cooldown_min = alert_cfg.get("cooldown_warning_minutes", 240)
        last = state.last_alerted_warning
    elif state.status == "critical":
        cooldown_min = alert_cfg.get("cooldown_critical_minutes", 60)
        last = state.last_alerted_critical
    else:
        return False
    if last is None:
        return True
    elapsed_min = (datetime.now(UTC) - datetime.fromisoformat(last)).total_seconds() / 60
    return elapsed_min >= cooldown_min
```

### Alert Format

```
🔴 METER CRITICAL

Signal:   Daily API Spend
Value:    $47.23
Critical: > $40.00
Change:   Warning → Critical

Time: 2026-06-03 14:30 ET
```

Recovery:
```
✅ METER HEALTHY

Signal:   QMD Index Age
Value:    0.0h
Status:   Recovered
Change:   Recovered — back to healthy

Time: 2026-06-03 22:39 ET
```

### ops-cache.json — meter section schema

```json
{
  "meter": {
    "last_run_iso": "2026-06-03T19:38:01+00:00",
    "last_run_at_ms": 1780511960792,
    "status": "warning",
    "signal_count": 18,
    "healthy_count": 15,
    "warning_count": 2,
    "critical_count": 1,
    "unknown_count": 0,
    "signals": [
      {
        "id": "cost.daily",
        "label": "Daily API Spend",
        "value": 30.23,
        "unit": "USD",
        "status": "warning",
        "threshold_warning": 15,
        "threshold_critical": 40,
        "trend": "down",
        "evaluated_at": "2026-06-03T19:38:01+00:00",
        "changed": false
      }
    ]
  }
}
```

---

## Consequences

### Benefits

**Alert quality over quantity.** State-transition alerting with cooldown windows means every alert represents a genuine change in system health. In practice: zero false-positive alerts after fixing one collector bug on day one.

**Config-driven extensibility.** Adding a new signal is 15 lines of YAML. No Python. No restart. Changing a threshold takes 30 seconds. This matters when you're calibrating thresholds post-deployment.

**Dashboard decoupling via ops-cache contract.** Any dashboard gets Meter data automatically. In the Cloud Nirvana implementation, integrating the Ops console Meter panel required one line added to the Bridge server and a new HTML panel. No API changes, no new endpoints.

**Collector failure isolation.** The first deployment had a bug — infrastructure collectors didn't accept `**kwargs`. Four signals became `unknown`. The other 14 collected and evaluated normally. The bug was visible and fixable without any service degradation.

**First alert validates the system.** The first real alert fired 30 minutes after deployment. It caught a real bug (QMD watcher loop dying after restart) that had been silently present for hours. The signal was legitimate. The investigation took 10 minutes. This is the pattern working as intended.

**Self-heal integration.** The QMD age signal detection was wired into the nightly maintenance cycle: if QMD age > 1h, run `openclaw memory index --force`. The platform now heals this known bug automatically at midnight.

### Tradeoffs

**30-minute polling is not real-time.** A cost spike starting at 14:00 may not alert until 14:30. For sub-minute alerting, use Prometheus + Grafana with a proper scrape interval instead.

**Trajectory file analysis is approximate.** Daily cost from trajectory files won't exactly match provider billing due to rounding, timezone differences, and sessions not yet flushed. Directionally correct; not auditable. For billing-grade accuracy, use the provider's usage API directly.

**Threshold calibration takes time.** Initial thresholds are educated guesses. Some will be too sensitive (firing on legitimate heavy sessions), some too loose (missing real anomalies). Plan for 2 weeks of baseline data before thresholds stabilize.

**Per-agent cost signals multiply signal count.** 5 per-agent cost signals doubles the cost signal set from 3 to 8. This is worth it for attribution — knowing "Radar is the expensive one today" is much more actionable than "total cost is high" — but it requires per-agent threshold calibration.

**State file is a single point of failure.** If `meter-state.json` is corrupted or deleted, all signals look like "first run" and may fire spurious transition alerts. Mitigate: keep the file small, back it up in the nightly maintenance cycle.

---

## Known Implementation Pitfalls

### 1. Measure log-based sync health, not file mtime
For background sync processes, measuring the mtime of the output artifact produces false positives. The QMD index file only updates when content changes — on a quiet day it's stale for hours even when the sync loop runs every 5 minutes. Measure the activity (log entries), not the artifact.

### 2. Threshold direction matters
Cost signals alert when value is *over* threshold. Cache efficiency alerts when value is *under* threshold. Both use the same mechanism; make `direction` explicit in your signal registry to avoid subtle bugs.

### 3. Python version compatibility
If your implementation environment runs Python 3.9, `float | None` union syntax won't work. Use `Optional[float]` from `typing`. Check your runtime version before writing collectors.

### 4. Gateway log file rotation
The log-based QMD collector reads the most recent 2 log files to handle overnight boundary conditions. If your platform rotates logs more frequently, increase this window. If logs aren't persisted at all, fall back to file mtime with a generous threshold.

### 5. Per-agent signals need per-agent baselines
A $20/day critical threshold makes sense for a lightweight operational agent. It's too low for a strategic agent doing heavy debug sessions. Set per-agent thresholds based on each agent's expected workload, not a uniform value.

---

## Known Uses

**Cloud Nirvana AIOS (2026-06-03):** Production deployment monitoring 18 signals across infrastructure, cost (aggregate + 5 per-agent), cron health, database, and memory. Built in response to a $176 single-day cost incident.

Current signal inventory:
- Infrastructure: gateway.rss, gateway.uptime, sessions.disk, sessions.count
- Cost (aggregate): cost.daily, cost.tokens_today, cost.cache_efficiency
- Cost (per-agent): cost.agent.main, cost.agent.radar, cost.agent.sales, cost.agent.events, cost.agent.speakers
- Cron health: cron.consecutive_errors, cron.error_rate
- Database: db.crm_size, db.hygiene_issues
- Memory: memory.qmd_index_age, memory.qmd_size

First real alert: QMD watcher loop bug. Caught 30 minutes after deployment. Fixed within the hour.

---

## Related Patterns

**[REM Cycle (Nightly Maintenance)](rem-cycle-nightly-maintenance.md):** Complementary. REM does deep nightly maintenance operations; Meter does continuous 30-minute monitoring. REM is episodic; Meter is continuous. Together they form a complete operational health loop: Meter detects, REM heals.

**[Escalation Chain with SLA](../operations-orchestration/escalation-chain-with-sla.md):** Meter critical alerts can feed an escalation chain. A critical signal that stays critical for N hours can auto-escalate beyond a Telegram alert.

**[System Hygiene for Agentic Systems](system-hygiene-for-agentic-systems.md):** System hygiene defines the practices that keep signals healthy. Meter tells you when hygiene has drifted.

---

## Practitioner Notes

*Real-world learnings from implementations. Add yours.*

**2026-06-03, Cloud Nirvana:** Full build log available. Key calibration insight: on the day we built this, the main session (our conversation) ran for 9 hours with heavy tool output and hit $30 of spend — well over the $20 critical threshold for `cost.agent.main`. This is legitimate work, not a runaway agent. The first threshold calibration task: raise `cost.agent.main` critical to $40-50 after seeing baseline data. The warning threshold ($8) is probably right for routine days.

**2026-06-03, Cloud Nirvana:** The `cache_write_5m` column in the Anthropic usage CSV was non-zero throughout the day until we applied `cacheRetention: long`. After the config change, `cache_write_5m` went to zero on every row. This is a clean validation signal that the cache retention change took effect. Consider tracking `cache_write_5m` as a signal in its own right — any non-zero value means you're not using long-term cache retention.

**2026-06-03, Cloud Nirvana:** The first false positive (QMD age warning) was caught by examining the signal carefully rather than just raising the threshold. The file mtime was stale but the sync loop was healthy. Lesson: when a new signal fires, diagnose the root cause before adjusting thresholds. Sometimes the alert is revealing a bug in your measurement, not in the system.

---

## Open Questions for Practitioners

1. **Threshold calibration methodology:** How do you determine initial thresholds for a new signal? Percentile-based (alert at 90th percentile of historical values)? Or start conservative and loosen after false positives?

2. **Provider usage API vs local trajectory analysis:** Have you connected to Anthropic/OpenAI usage APIs for real-time cost data? What's the lag? Is it reliable enough for 30-minute alerting?

3. **Historical trending storage:** How many readings do you retain before rotating? 12 readings (6h at 30m intervals) feels right for trend detection but may miss longer drift patterns.

4. **Multi-instance deployment:** If you run multiple gateway instances, how do you aggregate signals across them? Per-instance ops-cache files? A shared aggregator?

5. **Alert routing beyond Telegram:** Have you wired to PagerDuty, OpsGenie, or similar for on-call escalation? What's the right escalation path for a critical cost signal at 3 AM?

---

*This pattern was extracted from the Cloud Nirvana AIOS implementation.*
*Built 2026-06-03. Status: IN PROGRESS — production data accumulating.*
*Contributions welcome: github.com/cloudnirvana/open-patterns*
