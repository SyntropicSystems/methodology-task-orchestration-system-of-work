---
type: specification
category: observability
last_updated: 2025-10-18
---

# Reports Specification - Standard Reports

This specification defines standard reports generated from task management system data. Reports provide actionable insights for different audiences.

## Principles

1. **Actionable**: Each report drives specific decisions or improvements
2. **Consistent Format**: Standardized structure for easy consumption
3. **Event-Sourced**: All data derived from task ledgers and indexes
4. **Multiple Audiences**: DRIs, team leads, process improvement
5. **Trend-Aware**: Show direction of change, not just current state

## Report Catalog

### 1. Weekly Flow Report

**Audience**: Team leads, process improvement  
**Frequency**: Weekly (Monday morning)  
**Purpose**: System health overview, capacity planning

**Sections**:

#### A. Flow Summary
```
Week of: 2025-10-14 to 2025-10-20

Throughput: 15 tasks completed (↑ 3 from last week)
Arrival Rate: 18 tasks created (↑ 5 from last week)
Net Change: -3 tasks (demand > capacity)

WIP by Stage:
  inbox: 2 tasks
  backlog: 8 tasks
  next: 6 tasks
  current: 4 tasks (limit: 5)
  integration: 2 tasks
  learning: 1 task
  archived: 15 tasks (this week)

Total System WIP: 23 tasks (↑ 3 from last week)
```

#### B. Cycle Time Performance
```
Lead Time:
  Average: 9.5 days (target: <10 days)
  Median: 8.0 days
  P95: 18 days
  Trend: ↓ improving (was 10.2 days last week)

Stage Flow Times (averages):
  inbox: 6 hours (SLO: 24h) ✓
  backlog: 4.2 days (SLO: 7d) ✓
  current: 5.8 days (varies by effort) ~
  integration: 36 hours (SLO: 48h) ✓
  learning: 18 hours (SLO: 72h) ✓
  archived: 4 hours (SLO: 24h) ✓

Flow Efficiency: 58% (target: >40%) ✓
```

#### C. SLO Compliance
```
SLO Compliance Rate: 87% (target: >85%) ✓

SLO Misses This Week (2 tasks):
  TASK-0042: current stage (10d vs 5d target) - scope creep
  TASK-0051: integration stage (120h vs 48h target) - merge conflicts

Root Causes:
  - Scope creep: 1 task
  - Merge conflicts: 1 task
```

#### D. Quality Signals
```
Rework Rate: 6.7% (1 of 15 tasks) - within acceptable range
Rollback Events: 1 (TASK-0048: acceptance criteria changed)

P0 Activity: 1 P0 task this week
  - TASK-0055: API outage (resolved in 4 hours)
  - Displaced: TASK-0044 (paused, resumed after P0 completion)
```

#### E. Blockers Summary
```
Currently Blocked: 3 tasks
  TASK-0042: blocked by TASK-0010 (external API access) - 3 days
  TASK-0049: blocked by TASK-0023 (schema migration) - 1 day
  TASK-0050: blocked by external (vendor approval) - 5 days

Longest Blocked: TASK-0050 (5 days)
Top Blocker by Impact: TASK-0010 (blocking 2 tasks, 72 hours accumulated)
```

**Action Items**:
- Address TASK-0010 blocker (highest impact)
- Review P0 postmortem with team
- Investigate scope creep on TASK-0042

---

### 2. Blockers Report

**Audience**: DRIs, team leads  
**Frequency**: Daily or on-demand  
**Purpose**: Identify and resolve blocking tasks

**Format**:

```
=== BLOCKERS REPORT ===
Generated: 2025-10-18 09:00 AM

CURRENTLY BLOCKED TASKS (3)
────────────────────────────────────────────────────────────────
Task ID | Task Name              | Blocked By      | Duration | DRI
────────────────────────────────────────────────────────────────
0042    | User Authentication    | TASK-0010       | 3 days   | @alice
0049    | Payment Integration    | TASK-0023       | 1 day    | @bob
0050    | Analytics Dashboard    | external:vendor | 5 days   | @charlie
────────────────────────────────────────────────────────────────

TOP UPSTREAM BLOCKERS (by impact)
────────────────────────────────────────────────────────────────
Blocker ID | Blocker Name           | Blocking Count | Total Impact
────────────────────────────────────────────────────────────────
0010       | API Access Setup       | 2 tasks        | 72 hours
0023       | Database Schema Update | 1 task         | 24 hours
external   | Vendor Approval        | 1 task         | 120 hours
────────────────────────────────────────────────────────────────

RECOMMENDED ACTIONS:
1. Expedite TASK-0010 (API Access Setup) - highest blocker impact
2. Follow up on vendor approval (TASK-0050) - longest blocked time
3. Check TASK-0023 (Database Schema Update) - ready to integrate?
```

**Query Logic**:
```sql
-- Currently blocked tasks
SELECT task_id, task_name, blocked_by, 
       NOW() - (SELECT MAX(ts) FROM events WHERE event='blocked' AND task_id=t.task_id) AS blocked_duration,
       current_dri
FROM tasks t
WHERE status = 'blocked'
ORDER BY blocked_duration DESC;

-- Top blockers by impact
SELECT blocked_by AS blocker_id,
       COUNT(DISTINCT task_id) AS blocking_count,
       SUM(blocked_duration) AS total_impact
FROM (SELECT task_id, blocked_by, 
             NOW() - blocked_ts AS blocked_duration
      FROM events 
      WHERE event='blocked' AND task_id IN (SELECT task_id FROM tasks WHERE status='blocked'))
GROUP BY blocked_by
ORDER BY total_impact DESC;
```

---

### 3. Rework & Rollback Report

**Audience**: Process improvement, team leads  
**Frequency**: Weekly  
**Purpose**: Identify quality issues and workflow improvements

**Format**:

```
=== REWORK & ROLLBACK REPORT ===
Week of: 2025-10-14 to 2025-10-20

SUMMARY
─────────────────────────────────────
Total Completed: 15 tasks
Rollbacks: 1 task (6.7%)
Rework Rate Trend: ↓ improving (was 10% last week)

ROLLBACK DETAILS
─────────────────────────────────────────────────────────────────────
Task ID | Task Name           | Stage Rolled Back | Reason
─────────────────────────────────────────────────────────────────────
0048    | Payment Validation  | integration → current | Acceptance criteria changed
─────────────────────────────────────────────────────────────────────

COMMON ROLLBACK REASONS (Last 30 days)
─────────────────────────────────────────────────────────────────────
Reason                          | Count | % of Total
─────────────────────────────────────────────────────────────────────
Acceptance criteria changed     | 3     | 37.5%
Failed integration tests        | 2     | 25.0%
Incomplete implementation       | 2     | 25.0%
Merge conflicts                 | 1     | 12.5%
─────────────────────────────────────────────────────────────────────

INSIGHTS & RECOMMENDATIONS:
- Top cause: Acceptance criteria changes (37.5%)
  → Action: Implement WF-VALIDATE-MOVE checklist before moving to current
  → Action: Review acceptance criteria with stakeholders during WF3
  
- Failed integration tests (25%)
  → Action: Add pre-integration test checklist in WF5
  → Action: Consider local test environment for DRIs

REWORK RATE BY PRIORITY
─────────────────────────────────────
Priority | Rework Rate
─────────────────────────────────────
P0       | 0% (0 of 1)
P1       | 5% (1 of 20)
P2       | 8% (3 of 38)
P3       | 10% (2 of 20)
─────────────────────────────────────
```

**Query Logic**:
```sql
-- Rollback events
SELECT task_id, task_name, 
       (SELECT stage FROM events WHERE task_id=e.task_id AND ts < e.ts ORDER BY ts DESC LIMIT 1) AS rolled_back_from,
       desc AS reason
FROM events e
WHERE event = 'rollback'
  AND ts BETWEEN '2025-10-14' AND '2025-10-20';

-- Common reasons
SELECT desc AS reason, COUNT(*) AS count
FROM events
WHERE event = 'rollback'
  AND ts >= NOW() - INTERVAL '30 days'
GROUP BY reason
ORDER BY count DESC;
```

---

### 4. P0 Postmortem Summary

**Audience**: Team leads, process improvement  
**Frequency**: After each P0 task completion  
**Purpose**: Learn from critical incidents, improve P0 response

**Format**:

```
=== P0 POSTMORTEM ===
Task: TASK-0055 - API Outage Recovery
Completed: 2025-10-18 14:30
DRI: @alice

TIMELINE
─────────────────────────────────────────────────────────────────
Time              | Event
─────────────────────────────────────────────────────────────────
2025-10-18 10:15  | Outage detected, P0 task created
2025-10-18 10:20  | Task verified & prioritized (expedited)
2025-10-18 10:25  | Fast-tracked to current (skipped next stage)
2025-10-18 10:30  | TASK-0044 paused (displacement rule)
2025-10-18 10:35  | Root cause identified (database connection pool)
2025-10-18 12:00  | Fix implemented and tested
2025-10-18 12:15  | Deployed to production
2025-10-18 14:00  | Monitoring confirmed resolution
2025-10-18 14:30  | Task closed, TASK-0044 resumed
─────────────────────────────────────────────────────────────────

METRICS
─────────────────────────────────────────────────────────────────
Total Resolution Time: 4 hours 15 minutes
Time to Start (created → started): 10 minutes ✓
Time to Fix (started → implemented): 1 hour 30 minutes
Time to Deploy (implemented → integrated): 15 minutes
Displacement Impact: 1 task paused for 4 hours
─────────────────────────────────────────────────────────────────

ROOT CAUSE
Connection pool exhaustion due to recent traffic spike (3x normal load).
No auto-scaling configured for database connections.

PREVENTION MEASURES
1. Implement connection pool monitoring with alerts
2. Configure auto-scaling for database connection pool
3. Add load testing to integration checklist for database-heavy tasks

PROCESS LEARNINGS
✓ P0 fast-track worked well (10 min to start vs typical hours)
✓ Displacement rule executed smoothly (TASK-0044 resumed without issue)
✗ No monitoring alert fired before outage (reactive vs proactive)

ACTION ITEMS
[ ] @ops: Implement connection pool monitoring (TASK-0056)
[ ] @ops: Configure auto-scaling (TASK-0057)
[ ] @alice: Update integration checklist with load testing
[ ] @team: Review alert thresholds
```

**Required Data**:
- All ledger events for P0 task
- Displaced task information
- Stage flow times
- Root cause and prevention measures (from task card or learning doc)

---

### 5. Learning View Report

**Audience**: Process improvement  
**Frequency**: Monthly  
**Purpose**: Track learning capture, identify process improvement patterns

**Format**:

```
=== LEARNING VIEW REPORT ===
Month: October 2025

LEARNING COMPLETION RATE
─────────────────────────────────────────────────────────────────
Total Tasks Archived: 45
Learning Documents Created: 32 (71%)
Learning Skipped: 13 (29%)

Trend: Learning completion improving (was 65% last month) ↑
─────────────────────────────────────────────────────────────────

SKIP REASONS (13 tasks)
─────────────────────────────────────────────────────────────────
Reason                              | Count
─────────────────────────────────────────────────────────────────
Standard implementation             | 7
Duplicate of existing learning      | 4
Trivial task                        | 2
─────────────────────────────────────────────────────────────────

LEARNING PATTERN FREQUENCY (Top 5)
─────────────────────────────────────────────────────────────────
Pattern                                        | Count | Category
─────────────────────────────────────────────────────────────────
Underestimated complexity                      | 8     | Estimation
External dependencies cause delays             | 6     | Dependencies
Scope creep during implementation              | 5     | Scope Mgmt
Acceptance criteria clarified late            | 4     | Requirements
Integration testing reveals edge cases         | 3     | Quality
─────────────────────────────────────────────────────────────────

INSIGHTS
1. Estimation challenges persist (8 tasks underestimated)
   → Recommendation: Add buffer for new technology tasks
   
2. External dependencies causing significant delays (6 tasks)
   → Recommendation: Identify external deps in WF2 (verification)
   → Recommendation: Establish SLAs with external teams

3. Scope creep pattern (5 tasks)
   → Recommendation: Freeze scope when moving backlog → next
   → Recommendation: Use WF-SPLIT for new requirements discovered

TOP LEARNINGS (by impact)
─────────────────────────────────────────────────────────────────
TASK-0042: API authentication pattern for microservices (cited 3x)
TASK-0038: Database migration strategy for zero-downtime (cited 2x)
TASK-0035: Error handling patterns for async operations (cited 2x)
─────────────────────────────────────────────────────────────────

ACTION ITEMS
[ ] @team: Review top 3 learnings in next retrospective
[ ] @lead: Schedule estimation calibration workshop
[ ] @lead: Draft external dependency SLA proposal
```

**Query Logic**:
```sql
-- Learning completion
SELECT 
  COUNT(*) AS total,
  SUM(CASE WHEN event='learning_documented' THEN 1 ELSE 0 END) AS documented,
  SUM(CASE WHEN event='learning_skipped' THEN 1 ELSE 0 END) AS skipped
FROM events
WHERE event IN ('learning_documented', 'learning_skipped')
  AND ts BETWEEN '2025-10-01' AND '2025-10-31';

-- Pattern frequency (manual analysis of learning docs)
```

---

## Report Generation

### For Human Execution

**Weekly Flow Report**:
1. Parse all task ledgers for completed tasks this week
2. Count tasks in each stage from INDEX.md files
3. Compute metrics using formulas from `metrics.schema.md`
4. Compare to last week's report for trends
5. Format using template above

**Tools**: Spreadsheet, script, or manual tabulation

### For Automation

**Report Generator**:
```python
# Pseudocode
def generate_weekly_flow_report(start_date, end_date):
    # Query completed tasks
    completed = query.get_completed_tasks(start_date, end_date)
    throughput = len(completed)
    
    # Query WIP
    wip = query.get_wip_by_stage()
    
    # Compute metrics
    lead_times = [compute_lead_time(task) for task in completed]
    avg_lead_time = mean(lead_times)
    
    # Compute SLO compliance
    slo_misses = query.get_slo_misses(start_date, end_date)
    
    # Format report
    return format_flow_report(throughput, wip, avg_lead_time, slo_misses)
```

**Scheduled Execution**:
- Weekly reports: Monday 9:00 AM (automated email or posted to channel)
- Daily blockers report: Every morning 8:00 AM
- P0 postmortem: Triggered on P0 task `closed` event
- Monthly learning view: First Monday of each month

## Report Distribution

**Channels**:
- Email digest (weekly flow, monthly learning)
- Team chat (daily blockers, P0 postmortems)
- Dashboard (live WIP, current blockers)
- File-based (`.work/tasks/reports/YYYY-MM-DD-report-name.md`)

**Example File Structure**:
```
.work/tasks/reports/
├── 2025-10-14-weekly-flow.md
├── 2025-10-18-daily-blockers.md
├── 2025-10-18-p0-postmortem-0055.md
└── 2025-10-01-monthly-learning.md
```

## Report Customization

**Add Custom Reports**:
1. Define report purpose and audience
2. Specify data sources (ledgers, indexes, metrics)
3. Design report format (sections, tables, visualizations)
4. Document query logic
5. Add to report catalog

**Example Custom Report**: "Dependency Coupling Report"
- Purpose: Identify tasks with high dependency counts
- Audience: Architects, team leads
- Frequency: Monthly
- Query: Tasks with `depends_on` count >3

## References

- Metrics definitions: `metrics.schema.md`
- WIP limits and SLOs: `wip.policy.md`
- Query interface: `query.spec.md`
- Event schema: `events.schema.md`
- Workflow integration: `tasks.workflow.md`
