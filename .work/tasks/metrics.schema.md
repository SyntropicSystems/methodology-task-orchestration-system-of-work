---
type: schema
category: observability
last_updated: 2025-10-18
---

# Metrics Schema - Observability

This schema defines all metrics computed from the task management system's event logs. Metrics enable data-driven learning and continuous improvement.

## Principles

1. **Event-Sourced**: All metrics derived from immutable ledger events
2. **Point-in-Time**: Metrics computed as of a specific timestamp
3. **Aggregatable**: Can be summed across tasks, stages, time periods
4. **Actionable**: Each metric informs specific decisions or improvements

## Core Metrics

### Duration Metrics

**Lead Time** (end-to-end):
```
lead_time = closed_timestamp - created_timestamp
```
- Measures: Total time from task creation to archival
- Unit: Hours or days
- Purpose: Overall system efficiency
- Example: Task created 2025-10-01, closed 2025-10-15 → 14 days lead time

**Flow Time by Stage**:
```
stage_flow_time = exit_event_timestamp - entry_event_timestamp
```

| Stage | Entry Event | Exit Event | Typical Range |
|-------|-------------|------------|---------------|
| inbox | `created` | `verified` | <24 hours |
| backlog | `verified` | `prioritized` | <7 days |
| next | `prioritized` | `started` | 0 (on-demand) |
| current | `started` | `implemented` | 1-10 days |
| integration | `implemented` | `integrated` | <48 hours |
| learning | `integrated` | `learning_documented` or `learning_skipped` | <72 hours |
| archived | `learning_*` | `closed` | <24 hours |

**Blocked Time**:
```
blocked_time = SUM(unblocked_timestamp - blocked_timestamp) for all block periods
```
- Measures: Total time task was blocked
- Unit: Hours or days
- Purpose: Identify dependencies causing delays
- Example: Task blocked 3 times (2h, 5h, 1d) → 1d 7h blocked time

**Paused Time**:
```
paused_time = SUM(resumed_timestamp - paused_timestamp) for all pause periods
```
- Measures: Total time task was paused
- Unit: Hours or days
- Purpose: Understand P0 displacement impact or voluntary suspensions

**Active Time** (value-add time):
```
active_time = lead_time - blocked_time - paused_time
```
- Measures: Time task was actively progressing
- Unit: Hours or days
- Purpose: Flow efficiency calculation

### Flow Metrics

**Throughput**:
```
throughput = COUNT(closed events) / time_period
```
- Measures: Tasks completed per unit time
- Unit: Tasks per week (or day)
- Purpose: System capacity
- Example: 15 tasks closed in 1 week → 15 tasks/week throughput

**Arrival Rate**:
```
arrival_rate = COUNT(created events) / time_period
```
- Measures: New tasks entering system per unit time
- Unit: Tasks per week
- Purpose: Demand signal
- Example: 20 tasks created in 1 week → 20 tasks/week arrival rate

**Work-In-Progress (WIP)**:
```
wip = COUNT(tasks in stage with status = active) at point_in_time
```
- Measures: Active tasks in a stage at specific moment
- Unit: Count
- Purpose: Congestion detection, capacity planning
- Example: 5 tasks in `current` stage → WIP = 5

**Departure Rate** (by stage):
```
departure_rate = COUNT(exit events from stage) / time_period
```
- Measures: Tasks leaving a stage per unit time
- Unit: Tasks per week
- Purpose: Stage throughput, bottleneck identification

**Rework Rate**:
```
rework_rate = COUNT(rollback events) / COUNT(completed tasks) × 100%
```
- Measures: Percentage of tasks requiring rework
- Unit: Percentage
- Purpose: Quality signal, workflow effectiveness
- Example: 3 rollbacks out of 20 completions → 15% rework rate

### Derived Ratios

**Flow Efficiency**:
```
flow_efficiency = active_time / lead_time × 100%
```
- Measures: Percentage of lead time spent on value-add work
- Unit: Percentage
- Purpose: Waste identification (blocked time, wait time)
- Target: >40% (healthy), <20% (investigate)
- Example: 5 days active out of 14 days total → 36% flow efficiency

**Predictability** (Cycle Time Variability):
```
predictability = STDEV(lead_times) / AVG(lead_times)
```
- Measures: Consistency of delivery times (coefficient of variation)
- Unit: Ratio (lower is better)
- Purpose: Forecast reliability
- Target: <0.5 (predictable), >1.0 (highly variable)
- Example: Avg 10 days, StdDev 8 days → 0.8 (moderate variability)

**SLO Compliance**:
```
slo_compliance = (1 - COUNT(slo_missed events) / COUNT(completed tasks)) × 100%
```
- Measures: Percentage of tasks meeting SLO targets
- Unit: Percentage
- Purpose: Process reliability
- Target: >85%
- Example: 17 of 20 tasks met SLO → 85% compliance

**Dependency Ratio**:
```
dependency_ratio = AVG(COUNT(depends_on) per task)
```
- Measures: Average number of dependencies per task
- Unit: Count
- Purpose: Coupling detection
- Example: Tasks have 0, 1, 3, 2 dependencies → Avg 1.5

**Blocker Impact**:
```
blocker_impact = SUM(blocked_time for all tasks blocked by TASK-X)
```
- Measures: Total time other tasks blocked by a specific upstream task
- Unit: Hours or days
- Purpose: Identify critical path tasks
- Example: TASK-0010 blocks 3 tasks for 2d, 5d, 1d → 8 days blocker impact

## Metric Computation from Ledgers

### Event-to-Metric Mapping

**Example Ledger** (`0042.ledger.jsonl`):
```json
{"ts":"2025-10-01T10:00:00Z","user":"alice","event":"created","desc":"New feature request"}
{"ts":"2025-10-01T14:30:00Z","user":"bob","event":"verified","desc":"Package complete"}
{"ts":"2025-10-02T09:00:00Z","user":"bob","event":"prioritized","desc":"P2, moved to next"}
{"ts":"2025-10-03T10:00:00Z","user":"alice","event":"started","desc":"Implementation begun"}
{"ts":"2025-10-04T15:00:00Z","user":"alice","event":"blocked","desc":"Waiting for API access"}
{"ts":"2025-10-06T09:00:00Z","user":"alice","event":"unblocked","desc":"API credentials received"}
{"ts":"2025-10-08T16:00:00Z","user":"alice","event":"implemented","desc":"Code complete"}
{"ts":"2025-10-09T10:00:00Z","user":"bob","event":"integrated","desc":"Merged to main"}
{"ts":"2025-10-09T14:00:00Z","user":"alice","event":"learning_skipped","desc":"Standard implementation"}
{"ts":"2025-10-09T14:30:00Z","user":"bob","event":"closed","desc":"Task archived"}
```

**Computed Metrics**:
```
lead_time = 2025-10-09T14:30:00Z - 2025-10-01T10:00:00Z = 8.19 days
inbox_flow_time = 2025-10-01T14:30:00Z - 2025-10-01T10:00:00Z = 4.5 hours
backlog_flow_time = 2025-10-02T09:00:00Z - 2025-10-01T14:30:00Z = 18.5 hours
next_flow_time = 2025-10-03T10:00:00Z - 2025-10-02T09:00:00Z = 25 hours
current_flow_time = 2025-10-08T16:00:00Z - 2025-10-03T10:00:00Z = 5.25 days
blocked_time = 2025-10-06T09:00:00Z - 2025-10-04T15:00:00Z = 1.75 days
active_time = 8.19 - 1.75 = 6.44 days
flow_efficiency = 6.44 / 8.19 × 100% = 78.6%
```

### Aggregation Queries

**Weekly Throughput**:
```sql
SELECT COUNT(*) 
FROM events 
WHERE event = 'closed' 
  AND ts BETWEEN '2025-10-07' AND '2025-10-14'
```

**Average Lead Time (last 30 days)**:
```sql
SELECT AVG(closed_ts - created_ts) 
FROM tasks 
WHERE closed_ts BETWEEN NOW() - 30 days AND NOW()
```

**Blocked Tasks Report**:
```sql
SELECT task_id, blocked_by, 
       SUM(unblocked_ts - blocked_ts) AS total_blocked_time
FROM events 
WHERE event IN ('blocked', 'unblocked')
GROUP BY task_id
ORDER BY total_blocked_time DESC
```

## Time Series Metrics

Track trends over time to detect patterns and regime changes.

**Daily Metrics**:
- WIP by stage (point-in-time snapshot at EOD)
- Arrival rate (tasks created today)
- Departure rate (tasks closed today)

**Weekly Metrics**:
- Throughput (tasks closed this week)
- Average lead time (tasks closed this week)
- Rework rate (rollbacks / completions)
- SLO compliance (% meeting targets)

**Monthly Metrics**:
- Cumulative flow diagram (tasks entering/exiting each stage)
- Cycle time trends by priority level
- Blocker impact ranking (top 10 blocking tasks)

## Metric Categories by Audience

### For DRIs (Individual Performance)
- My WIP (tasks in `next` and `current` assigned to me)
- My average lead time (last 10 tasks)
- My blocked time percentage
- My learning completion rate

### For Team Leads (System Health)
- System WIP by stage
- Throughput vs arrival rate (capacity planning)
- SLO compliance by stage
- Top blockers (by blocker impact)
- Rework rate trends

### For Process Improvement (Double-Loop Learning)
- Flow efficiency trends
- Predictability (cycle time variability)
- Stage flow time distributions (identify bottlenecks)
- Dependency ratio (coupling)
- P0 frequency and displacement impact

## Metric Storage

**Option 1: Computed On-Demand** (human execution):
- Parse ledgers when generating reports
- No persistent storage
- Suitable for small-scale, infrequent queries

**Option 2: Pre-Aggregated** (automation):
- Daily batch job computes metrics
- Store in time-series database or CSV
- Fast querying, trend analysis
- Example storage: `.work/tasks/metrics/YYYY-MM-DD.metrics.json`

**Example Metrics File**:
```json
{
  "date": "2025-10-18",
  "system": {
    "wip_total": 8,
    "wip_by_stage": {"inbox": 2, "backlog": 5, "next": 4, "current": 3, "integration": 1},
    "throughput_weekly": 12,
    "arrival_rate_weekly": 15,
    "avg_lead_time_days": 9.5,
    "slo_compliance_pct": 87.5,
    "rework_rate_pct": 8.3
  },
  "flow_efficiency": {
    "avg_pct": 62.4,
    "p50_pct": 65.0,
    "p95_pct": 45.0
  },
  "top_blockers": [
    {"task_id": "0010", "blocker_impact_hours": 72},
    {"task_id": "0023", "blocker_impact_hours": 48}
  ]
}
```

## Event Vocabulary for Metrics

**Required events for core metrics**:
- `created` - Task creation (lead time start)
- `verified` - Inbox exit
- `prioritized` - Backlog exit
- `started` - Next exit, current entry
- `implemented` - Current exit
- `integrated` - Integration exit
- `learning_documented` or `learning_skipped` - Learning exit
- `closed` - Archived, lead time end
- `blocked` / `unblocked` - Blocked time tracking
- `paused` / `resumed` - Paused time tracking
- `rollback` - Rework detection
- `slo_missed` - SLO compliance

**All events must include**:
- `ts`: ISO 8601 timestamp with timezone
- `user`: User identifier
- `event`: Event type from vocabulary
- `desc`: Human-readable description

## Implementation Notes

**For Human Execution**:
- Use spreadsheet or script to parse `.ledger.jsonl` files
- Compute weekly metrics manually for flow reports
- Track WIP by counting entries in stage `INDEX.md` files

**For Automation**:
- Query layer (see `query.spec.md`) provides metric computation functions
- Worker contract (see `worker.contract.md`) enables automated metric collection
- Hooks (see `events.schema.md`) trigger metric updates on state changes

## References

- WIP limits and SLOs: `wip.policy.md`
- Standard reports: `reports.spec.md`
- Query interface: `query.spec.md`
- Event schema: `events.schema.md`
- Workflow integration: `tasks.workflow.md`
