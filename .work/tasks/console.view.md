---
type: specification
category: observability
last_updated: 2025-10-18
---

# Console Views - Operator Views

This specification defines standard console views that operators (humans and agents) use to understand system state at a glance. Views provide actionable situational awareness.

## Principles

1. **Glanceable**: Key information visible without scrolling
2. **Actionable**: Each view surfaces items requiring attention
3. **Real-Time**: Refresh frequently (or on-demand)
4. **Role-Based**: Different views for different audiences
5. **Drill-Down**: Summary → detail navigation

## View Catalog

### 1. Now View (Command Center)

**Audience**: Everyone  
**Purpose**: What needs attention right now  
**Refresh**: Real-time or every 30 seconds

**Layout**:
```
╔══════════════════════════════════════════════════════════════════╗
║                         NOW VIEW                                  ║
║                    Last Updated: 2025-10-18 10:30                ║
╠══════════════════════════════════════════════════════════════════╣
║ 🚨 ALERTS                                                         ║
╠══════════════════════════════════════════════════════════════════╣
║ • P0 Active: 1 task (FW-0055: API Outage Recovery)              ║
║ • WIP Limit: current stage at 5/5 (FULL)                        ║
║ • Stale Tasks: 2 tasks exceeding SLO (>2x target)               ║
╠══════════════════════════════════════════════════════════════════╣
║ 📊 WORK IN PROGRESS                                              ║
╠══════════════════════════════════════════════════════════════════╣
║ Stage       │ WIP │ Limit │ Status                               ║
║─────────────┼─────┼───────┼──────────────────────────────────────║
║ inbox       │   2 │  none │ 👍                                   ║
║ backlog     │   8 │  none │ 👍                                   ║
║ next        │   6 │  none │ 👍                                   ║
║ current     │   5 │     5 │ ⚠️  AT LIMIT                         ║
║ integration │   2 │     3 │ 👍                                   ║
║ learning    │   1 │     2 │ 👍                                   ║
╠══════════════════════════════════════════════════════════════════╣
║ 🔴 PENDING HUMAN SIGN-OFFS                                       ║
╠══════════════════════════════════════════════════════════════════╣
║ • FW-0042: Waiting for integration approval (@alice)             ║
║ • PLAT-0023: Waiting for learning sign-off (@bob)               ║
║ • MOBILE-0015: Waiting for move to current approval (@charlie)  ║
╠══════════════════════════════════════════════════════════════════╣
║ ⏸️  PAUSED TASKS (DISPLACED)                                     ║
╠══════════════════════════════════════════════════════════════════╣
║ • FW-0044: Paused for P0-0055 (4 hours ago)                     ║
╠══════════════════════════════════════════════════════════════════╣
║ 🔗 TOP BLOCKERS                                                  ║
╠══════════════════════════════════════════════════════════════════╣
║ • PLAT-0010: Blocking 3 tasks (72 hours accumulated)            ║
║ • FW-0023: Blocking 2 tasks (48 hours accumulated)              ║
╚══════════════════════════════════════════════════════════════════╝

Actions: [r]efresh | [d]etail | [h]elp | [q]uit
```

**Data Sources**:
- Alerts: Query P0 tasks, WIP limits, stale tasks
- WIP: `get_wip_by_stage()` from `query.spec.md`
- Sign-offs: Tasks in stages requiring human approval with status indicators
- Paused: Tasks with `status: paused` and `paused_reason` containing "P0"
- Blockers: `get_blocker_impact()` top 5

**Use Cases**:
- Morning standup dashboard
- Continuous display on team monitor
- Quick status check before starting work

---

### 2. Flow View (System Health)

**Audience**: Team leads, process improvement  
**Purpose**: Understand flow dynamics and capacity  
**Refresh**: Daily or on-demand

**Layout**:
```
╔══════════════════════════════════════════════════════════════════╗
║                       FLOW VIEW                                   ║
║                     Week of 2025-10-14                           ║
╠══════════════════════════════════════════════════════════════════╣
║ 📈 FLOW METRICS                                                  ║
╠══════════════════════════════════════════════════════════════════╣
║ Throughput:      15 tasks/week  ↑ +3 from last week             ║
║ Arrival Rate:    18 tasks/week  ↑ +5 from last week             ║
║ Net Change:      -3 tasks       ⚠️  demand > capacity           ║
║                                                                   ║
║ Lead Time:       9.5 days (avg)  ↓ improving                    ║
║ Flow Efficiency: 58%             👍 target >40%                  ║
║ SLO Compliance:  87%             👍 target >85%                  ║
╠══════════════════════════════════════════════════════════════════╣
║ 📊 CUMULATIVE FLOW (Last 4 Weeks)                               ║
╠══════════════════════════════════════════════════════════════════╣
║ 30│                                                              ║
║ 25│     ▁▂▃▅▆█ archived                                          ║
║ 20│   ▁▂▃▅▆██ learning                                           ║
║ 15│ ▁▂▃▅▆███ integration                                         ║
║ 10│▁▂▃▅█████ current                                             ║
║  5│▂▃▅██████ next                                                ║
║  0└────────────────────────────────────────────────             ║
║    W1   W2   W3   W4                                             ║
╠══════════════════════════════════════════════════════════════════╣
║ 🎯 STAGE FLOW TIMES (Averages)                                  ║
╠══════════════════════════════════════════════════════════════════╣
║ inbox       →  6 hours   (SLO: 24h)  ✓                          ║
║ backlog     →  4.2 days  (SLO: 7d)   ✓                          ║
║ current     →  5.8 days  (varies)    ~                          ║
║ integration →  36 hours  (SLO: 48h)  ✓                          ║
║ learning    →  18 hours  (SLO: 72h)  ✓                          ║
╠══════════════════════════════════════════════════════════════════╣
║ ⚠️  ISSUES                                                       ║
╠══════════════════════════════════════════════════════════════════╣
║ • Demand exceeding capacity (18 > 15 tasks/week)                ║
║ • Current stage at WIP limit (may cause queuing in next)        ║
║                                                                   ║
║ 💡 Recommendations:                                              ║
║   → Consider increasing current stage capacity                   ║
║   → Or reduce arrival rate (defer P3/P4 work)                   ║
╚══════════════════════════════════════════════════════════════════╝
```

**Data Sources**:
- Flow metrics: Computed from ledgers (see `metrics.schema.md`)
- Cumulative flow: WIP snapshots over time
- Stage flow times: Entry/exit event timestamps
- Issues: Compare arrival vs throughput, check WIP limits

**Use Cases**:
- Weekly retrospective
- Capacity planning
- Identifying bottlenecks

---

### 3. Blockers View (Dependency Map)

**Audience**: DRIs, team leads  
**Purpose**: Identify and resolve blockers  
**Refresh**: Real-time or hourly

**Layout**:
```
╔══════════════════════════════════════════════════════════════════╗
║                     BLOCKERS VIEW                                 ║
║                  Updated: 2025-10-18 10:30                       ║
╠══════════════════════════════════════════════════════════════════╣
║ 🔴 CURRENTLY BLOCKED (3 tasks)                                   ║
╠══════════════════════════════════════════════════════════════════╣
║ Task      │ Name               │ Blocked By  │ Duration │ DRI    ║
║───────────┼────────────────────┼─────────────┼──────────┼────────║
║ FW-0042   │ User Auth          │ PLAT-0010   │  3 days  │ @alice ║
║ FW-0049   │ Payment Gateway    │ PLAT-0023   │  1 day   │ @bob   ║
║ MOBILE-15 │ Analytics UI       │ external    │  5 days  │ @charlie║
╠══════════════════════════════════════════════════════════════════╣
║ ⛓️  TOP UPSTREAM BLOCKERS (by impact)                            ║
╠══════════════════════════════════════════════════════════════════╣
║ Blocker    │ Name               │ Count │ Total Impact           ║
║────────────┼────────────────────┼───────┼────────────────────────║
║ PLAT-0010  │ API Access Setup   │  3    │  72 hours              ║
║ PLAT-0023  │ Schema Migration   │  1    │  24 hours              ║
║ external   │ Vendor Approval    │  1    │ 120 hours              ║
╠══════════════════════════════════════════════════════════════════╣
║ 🎯 RECOMMENDED ACTIONS                                           ║
╠══════════════════════════════════════════════════════════════════╣
║ 1. Expedite PLAT-0010 (highest blocker impact)                  ║
║    → Status: In current stage                                    ║
║    → DRI: @bob                                                   ║
║    → Action: Check if ready to move to integration              ║
║                                                                   ║
║ 2. Follow up on vendor approval (MOBILE-0015)                   ║
║    → External blocker: 5 days                                    ║
║    → Action: Escalate to vendor liaison                          ║
║                                                                   ║
║ 3. Check PLAT-0023 status                                       ║
║    → Status: In integration stage                                ║
║    → Action: Verify integration complete                         ║
╠══════════════════════════════════════════════════════════════════╣
║ 📊 BLOCKER TRENDS                                                ║
╠══════════════════════════════════════════════════════════════════╣
║ This week:     3 blocked tasks  (↓ -1 from last week)           ║
║ Avg duration:  3.0 days         (↑ +0.5 from last week)         ║
║ Total impact:  216 hours        (↓ -24 from last week)          ║
╚══════════════════════════════════════════════════════════════════╝
```

**Data Sources**:
- Currently blocked: `get_blocked_tasks()` from `query.spec.md`
- Top blockers: `get_blocker_impact()` sorted by total impact
- Recommended actions: Heuristics based on blocker state
- Trends: Compare to previous week's blocker report

**Use Cases**:
- Daily blocker review
- Dependency coordination
- Escalation decisions

---

### 4. Rework View (Quality Signals)

**Audience**: Quality team, process improvement  
**Purpose**: Track rework patterns and quality issues  
**Refresh**: Weekly or on-demand

**Layout**:
```
╔══════════════════════════════════════════════════════════════════╗
║                      REWORK VIEW                                  ║
║                   Week of 2025-10-14                             ║
╠══════════════════════════════════════════════════════════════════╣
║ 📊 REWORK SUMMARY                                                ║
╠══════════════════════════════════════════════════════════════════╣
║ Completed:     15 tasks                                          ║
║ Rollbacks:      1 task   (6.7%)                                  ║
║ Trend:          ↓ improving (was 10% last week)                  ║
╠══════════════════════════════════════════════════════════════════╣
║ 🔄 ROLLBACK DETAILS                                              ║
╠══════════════════════════════════════════════════════════════════╣
║ Task      │ Name           │ Stage Rolled Back      │ Reason     ║
║───────────┼────────────────┼────────────────────────┼────────────║
║ FW-0048   │ Payment Valid  │ integration → current  │ Acceptance  ║
║           │                │                        │ criteria   ║
║           │                │                        │ changed    ║
╠══════════════════════════════════════════════════════════════════╣
║ 📈 COMMON ROLLBACK REASONS (Last 30 Days)                       ║
╠══════════════════════════════════════════════════════════════════╣
║                                                                   ║
║    Acceptance criteria changed  ████████░░░  37.5%  (3 tasks)   ║
║    Failed integration tests     ██████░░░░░  25.0%  (2 tasks)   ║
║    Incomplete implementation    ██████░░░░░  25.0%  (2 tasks)   ║
║    Merge conflicts              ███░░░░░░░░  12.5%  (1 task)    ║
║                                                                   ║
╠══════════════════════════════════════════════════════════════════╣
║ 💡 INSIGHTS & RECOMMENDATIONS                                    ║
╠══════════════════════════════════════════════════════════════════╣
║ Top Issue: Acceptance criteria changes (37.5%)                   ║
║   → Action: Implement WF-VALIDATE-MOVE checklist                 ║
║   → Action: Review criteria with stakeholders in WF3             ║
║                                                                   ║
║ Failed integration tests (25%)                                   ║
║   → Action: Add pre-integration test checklist in WF5            ║
║   → Action: Consider local test environment for DRIs             ║
╠══════════════════════════════════════════════════════════════════╣
║ 🎯 REWORK RATE BY PRIORITY                                       ║
╠══════════════════════════════════════════════════════════════════╣
║ P0: ░░░░░░░░░░  0% (0 of 1)                                      ║
║ P1: █░░░░░░░░░  5% (1 of 20)                                     ║
║ P2: ██░░░░░░░░  8% (3 of 38)                                     ║
║ P3: ██░░░░░░░░ 10% (2 of 20)                                     ║
╚══════════════════════════════════════════════════════════════════╝
```

**Data Sources**:
- Rework summary: `get_rework_history()` from `query.spec.md`
- Rollback details: Parse ledgers for `rollback` events
- Common reasons: Frequency analysis of rollback descriptions
- Rework rate by priority: Group rollbacks by priority level

**Use Cases**:
- Quality retrospectives
- Process improvement planning
- Training needs identification

---

### 5. Learning View (Process Evolution)

**Audience**: Process improvement team  
**Purpose**: Track learning capture and patterns  
**Refresh**: Monthly

**Layout**:
```
╔══════════════════════════════════════════════════════════════════╗
║                     LEARNING VIEW                                 ║
║                    Month: October 2025                           ║
╠══════════════════════════════════════════════════════════════════╣
║ 📚 LEARNING COMPLETION RATE                                      ║
╠══════════════════════════════════════════════════════════════════╣
║ Total Archived:      45 tasks                                    ║
║ Learning Created:    32 tasks (71%)                              ║
║ Learning Skipped:    13 tasks (29%)                              ║
║                                                                   ║
║ Trend: ↑ improving (was 65% last month)                         ║
║                                                                   ║
║ Progress: ███████████████████████░░░░░░░░░░  71%                ║
╠══════════════════════════════════════════════════════════════════╣
║ 🏷️  SKIP REASONS                                                 ║
╠══════════════════════════════════════════════════════════════════╣
║ Standard implementation     ███████░░░  7 tasks                  ║
║ Duplicate learning          ████░░░░░░  4 tasks                  ║
║ Trivial task                ██░░░░░░░░  2 tasks                  ║
╠══════════════════════════════════════════════════════════════════╣
║ 🔍 LEARNING PATTERN FREQUENCY (Top 5)                           ║
╠══════════════════════════════════════════════════════════════════╣
║ Pattern                           │ Count │ Category             ║
║───────────────────────────────────┼───────┼──────────────────────║
║ Underestimated complexity         │   8   │ Estimation           ║
║ External deps cause delays        │   6   │ Dependencies         ║
║ Scope creep during impl           │   5   │ Scope Mgmt           ║
║ Acceptance criteria late          │   4   │ Requirements         ║
║ Integration reveals edge cases    │   3   │ Quality              ║
╠══════════════════════════════════════════════════════════════════╣
║ 💡 INSIGHTS                                                      ║
╠══════════════════════════════════════════════════════════════════╣
║ 1. Estimation challenges persist (8 tasks)                       ║
║    → Recommendation: Add buffer for new technology tasks         ║
║                                                                   ║
║ 2. External dependencies causing delays (6 tasks)               ║
║    → Recommendation: Identify external deps in WF2               ║
║    → Recommendation: Establish SLAs with external teams          ║
║                                                                   ║
║ 3. Scope creep pattern (5 tasks)                                ║
║    → Recommendation: Freeze scope when moving backlog → next     ║
║    → Recommendation: Use WF-SPLIT for new requirements           ║
╠══════════════════════════════════════════════════════════════════╣
║ 🌟 TOP LEARNINGS (by impact/citations)                          ║
╠══════════════════════════════════════════════════════════════════╣
║ • FW-0042: API auth pattern for microservices (cited 3x)        ║
║ • FW-0038: Database migration for zero-downtime (cited 2x)      ║
║ • FW-0035: Error handling for async operations (cited 2x)       ║
╚══════════════════════════════════════════════════════════════════╝
```

**Data Sources**:
- Learning completion: Count `learning_documented` vs `learning_skipped` events
- Skip reasons: Parse `learning_skipped` event descriptions
- Pattern frequency: Manual analysis of `{id}.learning.md` files
- Top learnings: Track references to learning docs in other tasks

**Use Cases**:
- Monthly learning retrospective
- Process improvement identification
- Knowledge management

---

## View Implementation

### Option 1: Terminal UI (TUI)

**Tech Stack**: Python + `rich` or `textual` library

**Example**:
```python
from rich.console import Console
from rich.table import Table
from rich.live import Live
import time

def now_view():
    """Render Now View in terminal."""
    console = Console()
    
    with Live(console=console, refresh_per_second=0.5) as live:
        while True:
            # Fetch data
            p0_tasks = get_p0_tasks()
            wip = get_wip_by_stage()
            sign_offs = get_pending_sign_offs()
            
            # Build table
            table = Table(title="NOW VIEW")
            table.add_column("Stage")
            table.add_column("WIP")
            table.add_column("Limit")
            table.add_column("Status")
            
            for stage, count in wip.items():
                limit = get_wip_limit(stage)
                status = "⚠️ AT LIMIT" if count >= limit else "👍"
                table.add_row(stage, str(count), str(limit), status)
            
            live.update(table)
            time.sleep(30)  # Refresh every 30 seconds
```

### Option 2: Web Dashboard

**Tech Stack**: React + Next.js + real-time WebSocket updates

**Features**:
- Multiple views as tabs
- Auto-refresh with visual indicators
- Drill-down to task details
- Mobile-responsive

### Option 3: CLI Commands

**Simple text output for automation**:
```bash
# Now view
task-cli view now

# Flow view
task-cli view flow

# Blockers view
task-cli view blockers --format json
```

### Option 4: Monitor Display

**Large screen for team area**:
- Rotate through views every 30 seconds
- Large fonts, high contrast
- Sound alerts for P0 or WIP limit violations

## View Customization

### User-Defined Filters

**Example**: Personal view showing only my tasks
```bash
task-cli view now --filter "dri:@alice"
```

**Example**: Namespace-specific view
```bash
task-cli view flow --namespace FW
```

### Custom Views

**Define custom view** in `.work/tasks/.views/custom.yaml`:
```yaml
my_team_view:
  title: "My Team Dashboard"
  sections:
    - type: wip
      filter: "dri:@alice OR dri:@bob"
    - type: blockers
      filter: "namespace:FW"
    - type: sign_offs
      filter: "assigned_to:@alice"
```

## Integration with Workflows

### Workflow Decision Points

**Before WF4 (move to current)**:
- Check Now View → WIP limit status
- If at limit, pause or complete existing work first

**During Daily Standup**:
- Review Now View → Alerts, sign-offs, blockers
- Discuss Flow View → Throughput trends

**Weekly Retrospective**:
- Review Flow View → Metrics and trends
- Review Rework View → Quality patterns
- Review Learning View → Process improvements

## References

- Query interface: `query.spec.md`
- Metrics schema: `metrics.schema.md`
- Reports specification: `reports.spec.md`
- WIP policy: `wip.policy.md`
- Worker contract: `worker.contract.md`
