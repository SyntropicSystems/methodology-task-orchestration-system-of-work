---
type: policy
category: flow_control
last_updated: 2025-10-18
---

# WIP Policy - Flow Control

This policy defines Work-In-Progress (WIP) limits, Service Level Objectives (SLOs), queue discipline, and violation handling to enable safe speed and predictable flow.

## Principles

1. **Little's Law**: WIP = Throughput × Lead Time
   - Lower WIP → faster flow, shorter lead times
   - Higher WIP → congestion, unpredictability, context switching

2. **Theory of Constraints**: Optimize the bottleneck, subordinate everything else
   - Identify limiting stage (usually `current`)
   - Set WIP limits to match constraint capacity

3. **Stop the Line (Andon)**: Quality over speed
   - P0 issues trigger immediate attention
   - System-wide coordination to resolve critical blockers

## WIP Limits by Stage

| Stage | WIP Limit | Scope | Rationale |
|-------|-----------|-------|-----------|
| inbox | None | System-wide | Entry point, no artificial constraint |
| backlog | None | System-wide | Infinite queue, prioritization determines pull |
| next | ≤3 | Per DRI | Individual capacity planning (ready but not started) |
| current | ≤5 | System-wide | Constraint: active implementation capacity |
| integration | ≤3 | System-wide | Technical merge + release capacity |
| learning | ≤2 | Per DRI | Double-loop reflection requires focus |
| archived | None | System-wide | Final resting place |

### WIP Limit Enforcement

**Hard Limits** (blocking):
- **current**: Cannot exceed 5 active tasks system-wide
  - Violation: `WF4` (next → current) fails with error
  - Action: Must complete or pause existing work before starting new tasks

**Soft Limits** (guidance):
- **next (per DRI)**: Recommended ≤3 tasks per person
  - Violation: Warning logged, not blocking
  - Action: DRI should reprioritize or delegate

- **integration**: Recommended ≤3 tasks
  - Violation: Warning logged, not blocking
  - Action: Expedite integration to clear queue

- **learning**: Recommended ≤2 tasks per DRI
  - Violation: Warning logged, not blocking
  - Action: Complete learning documents or skip learning stage

### P0 Exception (Andon Rule)

**When P0 arrives**:
1. **Immediate attention**: P0 bypasses `next` stage (fast-track to `current`)
2. **Displacement rule**: If `current` at WIP limit (5 tasks):
   - Lowest priority non-P0 task is **paused** (not canceled)
   - Log displacement: `{"event":"displaced_by_p0","desc":"Paused TASK-XXXX to accommodate P0 TASK-YYYY"}`
   - Displaced task remains in `current` with `status: paused`
3. **P0 hygiene obligation**: When P0 completes:
   - DRI must explicitly **resume** at least one displaced task
   - If no displaced tasks exist, proceed normally

**P0 Coordination Protocol**:
```
1. Create P0 task in inbox
2. Announce in team chat: "P0 TASK-XXXX created: [brief description]"
3. Verify + prioritize immediately (WF2 + WF3 expedited)
4. Apply displacement rule if current stage at capacity
5. Execute P0 work with full team awareness
6. Post-mortem: Log P0 learning in task ledger
7. Resume displaced work explicitly
```

## Queue Discipline

**Priority Ordering**:
- Within each stage, tasks are ordered by priority (P0 > P1 > P2 > P3 > P4)
- Within same priority: First-In-First-Out (FIFO)

**Exception - Rework**:
- Rework tasks (those with `rollback` event in ledger) use **Last-In-First-Out (LIFO)**
- Rationale: Fresh context, shorter cycle time for rework

**Pull System**:
- Tasks are **pulled** from upstream stage, not pushed
- DRI initiates `WF4` (next → current) when capacity available
- Backlog refill happens via `WF3` (backlog → next) based on priority

## Service Level Objectives (SLOs)

SLOs define target durations for each stage. These are **guidance**, not hard constraints.

| Stage | SLO Target | Measurement | Violation Threshold |
|-------|-----------|-------------|---------------------|
| inbox | 24 hours | Time from `created` to `verified` | >48 hours |
| backlog | 7 days | Time from `verified` to `prioritized` | >14 days |
| next | Immediate | Time from `prioritized` to `started` (on-demand pull) | N/A (demand-driven) |
| current | Varies by effort | Time from `started` to `implemented` | >2x effort estimate |
| integration | 48 hours | Time from `implemented` to `integrated` | >96 hours |
| learning | 72 hours | Time from `integrated` to `learning_documented` or `learning_skipped` | >7 days |
| archived | 24 hours | Time from `learning_*` to `closed` | >48 hours |

### SLO Violation Handling

**When SLO missed**:
1. Log event in task ledger:
   ```json
   {"ts":"ISO8601","user":"system","event":"slo_missed","desc":"Stage: current, Target: 5d, Actual: 12d, Reason: external dependency delay"}
   ```

2. DRI investigates root cause:
   - External blocker? → Log `blocked` event with reason
   - Underestimated effort? → Update effort estimate, log learning
   - Process inefficiency? → Propose workflow improvement

3. Include in Weekly Flow Report (see `reports.spec.md`)

4. **No penalties**: SLOs are learning signals, not performance metrics

### SLO Computation

**Formula**:
```
stage_duration = timestamp(exit_event) - timestamp(entry_event)
```

**Examples**:
- inbox SLO: `verified` timestamp - `created` timestamp
- current SLO: `implemented` timestamp - `started` timestamp

**Data source**: Task ledgers (`.ledger.jsonl` files)

## Flow Control Signals

### Yellow Flags (Warnings)
- WIP approaching limit (e.g., current at 4/5 tasks)
- Task in stage >75% of SLO target
- Multiple tasks blocked by same upstream dependency

**Action**: Monitor, prepare to intervene

### Red Flags (Violations)
- WIP limit exceeded (current >5 tasks)
- SLO missed by >2x threshold
- Cycle detected in dependency graph

**Action**: Stop and resolve immediately

### Green Signals (Healthy Flow)
- WIP below limits with steady throughput
- SLOs consistently met
- No blocked tasks or blockers resolving quickly

**Action**: Continue current practices

## Capacity Planning

**Individual Capacity**:
- **next stage**: Each DRI should have ≤3 tasks ready (not started)
- **current stage**: DRI typically works on 1-2 tasks concurrently
- **learning stage**: DRI should complete learning for ≤2 tasks at a time

**System Capacity**:
- **current stage**: 5 active tasks system-wide (hard limit)
- Adjust based on team size and measured throughput
- Review WIP limits quarterly based on flow metrics

**Capacity Formula** (example):
```
system_throughput = tasks_completed / week
average_lead_time = total_time_in_system / tasks_completed
optimal_wip = system_throughput × average_lead_time
```

Use metrics from `metrics.schema.md` to compute and adjust.

## Event Logging

**WIP-related events**:
```json
{"ts":"ISO8601","user":"user_id","event":"wip_limit_approached","desc":"current stage at 4/5 capacity"}
{"ts":"ISO8601","user":"user_id","event":"wip_limit_blocked","desc":"Cannot start TASK-XXXX: current stage at capacity (5/5)"}
{"ts":"ISO8601","user":"user_id","event":"displaced_by_p0","desc":"Paused TASK-XXXX to accommodate P0 TASK-YYYY"}
{"ts":"ISO8601","user":"user_id","event":"slo_missed","desc":"Stage: current, Target: 5d, Actual: 12d, Reason: scope creep"}
```

## Implementation Notes

**For Human Execution**:
- Before executing `WF4` (next → current), manually count active tasks in `current/INDEX.md`
- If count = 5, either complete existing task or pause lowest priority task
- Log WIP violations in system ledger or team log

**For Automation**:
- Query layer (see `query.spec.md`) provides `get_stage_wip(stage)` function
- WIP enforcement hook in `events.schema.md`: `on_validate_move`
- SLO monitoring daemon computes durations from ledgers, emits alerts

## References

- Metrics computation: `metrics.schema.md`
- Standard reports: `reports.spec.md`
- Query interface: `query.spec.md`
- Event hooks: `events.schema.md`
- Workflow integration: `tasks.workflow.md`
