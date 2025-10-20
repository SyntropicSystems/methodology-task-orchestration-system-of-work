---
type: workflow
stage: system
parent: ../tasks.workflow.md
revisit_after_executions: 25
last_revisited: 2025-10-18
criticality: flexible
---

# Priority & Effort Workflows

This file contains workflows for managing task priority (P0-P4) and effort estimation (XS-XL).

## P0 Fast-Track Rules

**Criticality**: must (for P0 tasks)

**Purpose**: Define expedited workflow for critical priority tasks that require immediate attention.

**When**: Task is assigned `priority: P0` (Critical)

**P0 Criteria** (must meet ALL):
- Production system down or severely degraded
- Active security breach or critical vulnerability
- Data loss or corruption occurring
- User-facing critical failure affecting majority of users
- No workaround exists

**Stage Path** (default):
```
inbox → backlog (verification) → current
```
- **Skips**: `next` stage
- **Required**: WF2 verification still applies (task must be properly formed)
- **Required**: WF-VALIDATE-MOVE still runs before entering current

**Steps**:

1. **Assign P0 Priority** ⚠️ **MUST**
   - Set `priority: P0` in task frontmatter
   - Document why this is P0 (meets criteria above)
   - Log:
     ```json
     {"ts":"ISO8601","user":"user_id","event":"priority_set","value":"P0","reason":"[critical situation description]"}
     ```
   - Why MUST: P0 requires justification

2. **Verify Task Package** ⚠️ **MUST**
   - Still execute WF2 verification (expedited, ~5 min max)
   - Ensure task card has clear goal, acceptance criteria
   - Verify DRI is assigned and available
   - Why MUST: Even urgent work needs clarity

3. **Move to Current** ⚠️ **MUST**
   - Skip `next` stage, move directly from `backlog` → `current`
   - Execute WF-VALIDATE-MOVE before move
   - If dependencies not complete:
     - Option A: Mark as `blocked` and address dependencies first
     - Option B: Log `concurrent_ok` waiver if risk is acceptable
   - Log:
     ```json
     {"ts":"ISO8601","user":"user_id","event":"fast_tracked","desc":"P0 moved directly to current, skipped next stage","reason":"[justification]"}
     ```
   - Why MUST: Fast-track requires audit trail

4. **Handle Current Stage Displacement** ⚠️ **MUST**
   
   **If current stage has active work:**
   - Choose one task to pause (typically lowest priority, or least urgent P1-P3)
   - Execute WF-PAUSE on that task
   - Move paused task back to `next`
   - Log in both tasks:
     ```json
     # In paused task:
     {"ts":"ISO8601","user":"user_id","event":"paused_by_p0","desc":"Preempted by P0 TASK-XXXX","preempted_by":"TASK-XXXX"}
     
     # In P0 task:
     {"ts":"ISO8601","user":"user_id","event":"displaced","desc":"Displaced TASK-YYYY to accommodate P0 work","displaced_task":"TASK-YYYY"}
     ```
   - Notify displaced task DRI immediately
   
   **If multiple P0 tasks:**
   - Work in parallel if possible (different DRIs, different systems)
   - If same DRI or same system: sequence by impact severity
   - Log sequencing decision
   
   Why MUST: Resource conflicts must be resolved explicitly

5. **Implement with Urgency** ⚠️ **MUST**
   - Work proceeds in `current` stage
   - Regular check-ins recommended (hourly or more frequent)
   - Document progress in context file frequently
   - Bypass normal review cycles if necessary (document waivers)
   - Why MUST: P0 requires rapid iteration

6. **Post-Facto Hygiene** ⚠️ **MUST**
   - Within 72 hours of P0 completion:
     - Create learning document (`{id}.learning.md`) analyzing:
       - What caused the P0 situation
       - How it was resolved
       - How to prevent recurrence
     - Clear any temporary waivers or shortcuts taken
     - Create follow-up tasks for any technical debt incurred
   - Log:
     ```json
     {"ts":"ISO8601","user":"user_id","event":"p0_retrospective","desc":"Post-mortem completed, follow-ups created"}
     ```
   - Why MUST: P0s are learning opportunities

**P0 Exceptions**:

- **Integration**: Cannot be skipped - P0 changes must still be properly integrated and released
- **Learning**: Can be deferred but must complete within 72 hours
- **Dependencies**: Can log `concurrent_ok` waiver but must document risks

**Required Ledger Events**:
- Priority assignment: `{"ts":"ISO8601","user":"user_id","event":"priority_set","value":"P0","reason":"[justification]"}`
- Fast-track: `{"ts":"ISO8601","user":"user_id","event":"fast_tracked","desc":"P0 moved to current, skipped next"}`
- Displacement (if any): `{"ts":"ISO8601","user":"user_id","event":"paused_by_p0","desc":"Preempted by P0 TASK-XXXX"}`
- Retrospective: `{"ts":"ISO8601","user":"user_id","event":"p0_retrospective","desc":"Post-mortem completed"}`

**P0 Fast-Track Checklist**:
- [ ] P0 criteria verified (meets all requirements)
- [ ] Task package verified (WF2 expedited)
- [ ] Priority set and logged
- [ ] Dependencies validated or waived
- [ ] Current stage displacement handled (if needed)
- [ ] Fast-track event logged
- [ ] DRI committed and available
- [ ] Regular check-ins scheduled
- [ ] Post-facto retrospective scheduled (within 72h)

---

## WF-SET-PRIORITY - Assign Priority

**Criticality**: flexible

**Purpose**: Assign a priority level (P0-P4) to a task during backlog grooming.

**When**: During backlog stage, before moving task to next. Required before a task can be prioritized.

**Priority Levels**:
- **P0**: Critical (system down, security breach) - bypasses normal flow
- **P1**: High (high-value, time-sensitive) - primary focus
- **P2**: Medium (standard work) - default priority
- **P3**: Low (nice-to-have) - when capacity available
- **P4**: Defer (acknowledged but not planned) - remains in backlog

**Steps**:

1. **Review Task** ⚠️ **MUST**
   - Read task Goal and Acceptance Criteria
   - Understand business value and urgency
   - Consider dependencies and blockers
   - Why MUST: Informed priority decision requires context

2. **Assess Priority** ✓ Flexible
   - Evaluate against priority schema
   - Consider:
     - Business impact
     - Time sensitivity
     - Dependencies (does this block other work?)
     - Strategic importance
     - Risk if delayed
   - Skip reason example: "Obvious priority, documented in ledger only"

3. **Assign P-Level** ⚠️ **MUST**
   - Choose appropriate level: P0, P1, P2, P3, or P4
   - Default to P2 if uncertain
   - Why MUST: Task cannot move to next without priority

4. **Update Task Frontmatter** ⚠️ **MUST**
   - Set `priority: "P2"` (or appropriate level)
   - Why MUST: Single source of truth

5. **Update Index** ✓ Flexible
   - Optionally add priority indicator to task name or flags
   - Skip reason example: "Priority visible in frontmatter, no need in index"

6. **Log Event** ⚠️ **MUST**
   - Add to ledger: `{"ts":"ISO8601","user":"user_id","event":"priority_set","value":"P2","reason":"standard feature work, normal business value"}`
   - Why MUST: Audit trail of priority decisions

**P0 Special Handling**:

If assigning **P0** (Critical):
- Task should skip backlog and next stages
- Move directly from inbox → current (after verification)
- Any P1-P3 task in current may need to be paused
- Log displacement: `{"ts":"ISO8601","user":"user_id","event":"task_displaced","displaced_task":"TASK-0042","reason":"P0 critical work takes precedence"}`

**Priority Assignment Guidelines**:

| Level | Use When | Examples |
|-------|----------|----------|
| P0 | Production down, security breach, critical blocker | "API gateway down, all users blocked" |
| P1 | High business value, time-sensitive deadline | "Launch feature for paid tier by Q4" |
| P2 | Standard feature work, normal cadence | "Refactor settings page", "Add export feature" |
| P3 | Nice-to-have improvements, fill-in work | "Optimize admin script", "Polish UI animation" |
| P4 | Acknowledged but deprioritized, no plan | "Support legacy edge case", "Obscure feature request" |

**Skip Guidance**:
- **MUST steps**: Priority assignment and logging are required
- **Flexible steps**: Assessment depth can adapt to context

**Required Ledger Event**:
```json
{"ts":"2025-10-18T10:30:00Z","user":"@alice","event":"priority_set","value":"P2","reason":"standard feature work with normal business value"}
```

---

## WF-CHANGE-PRIORITY - Update Priority

**Criticality**: flexible

**Purpose**: Change a task's priority when new information emerges or context changes.

**When**:
- Business priorities shift
- Deadline moves (earlier or later)
- Dependencies change
- New information about impact or urgency
- Strategic pivot

**Steps**:

1. **Document Reason for Change** ⚠️ **MUST**
   - Record specific reason for priority change
   - This creates audit trail of decision-making
   - Why MUST: Priority changes are significant decisions

2. **Determine New Priority** ⚠️ **MUST**
   - Assess new priority level using same criteria as WF-SET-PRIORITY
   - Consider impact of the change on other work
   - Why MUST: Informed decision required

3. **Update Task Frontmatter** ⚠️ **MUST**
   - Change `priority` field from old to new value
   - Why MUST: Single source of truth

4. **Update Indexes** ✓ Flexible
   - Update any priority indicators in index entries
   - Skip reason example: "Priority stored in frontmatter, sufficient"

5. **Log Event** ⚠️ **MUST**
   - Add to ledger: `{"ts":"ISO8601","user":"user_id","event":"priority_changed","from":"P2","to":"P1","reason":"deadline moved up by two weeks"}`
   - Why MUST: Complete audit trail

6. **Handle Stage Transitions** ⚠️ **MUST** (if changing to/from P0)
   - **Upgrading to P0**: If task not in current, consider moving to current immediately
   - **Downgrading from P0**: May need to pause and return to next
   - Log any stage movements
   - Why MUST: P0 has special handling

7. **Review Impact on Other Tasks** ✓ Flexible
   - If priority increased significantly, consider if it affects sequencing
   - If priority decreased, consider if blocked tasks should be reprioritized
   - Skip reason example: "No impact on other work"

**Priority Change Scenarios**:

**Scenario 1: Deadline Advancement**
```json
{"ts":"2025-10-18T10:30:00Z","user":"@alice","event":"priority_changed","from":"P2","to":"P1","reason":"customer deadline moved from Q1 to Q4, now time-sensitive"}
```

**Scenario 2: Requirement Obsolescence**
```json
{"ts":"2025-10-18T10:30:00Z","user":"@alice","event":"priority_changed","from":"P1","to":"P4","reason":"strategic pivot, feature no longer critical"}
```

**Scenario 3: Critical Issue**
```json
{"ts":"2025-10-18T10:30:00Z","user":"@alice","event":"priority_changed","from":"P1","to":"P0","reason":"production bug affecting 80% of users"}
```

**Skip Guidance**:
- **MUST steps**: Documentation and logging maintain decision audit trail
- **Flexible steps**: Impact analysis adapts to severity of change

**Required Ledger Event**:
```json
{"ts":"ISO8601","user":"user_id","event":"priority_changed","from":"P2","to":"P1","reason":"[specific reason for change]"}
```

---

## WF-ESTIMATE - Assign Effort Estimate

**Criticality**: flexible

**Purpose**: Assign an effort estimate to a task for capacity planning.

**When**: During backlog grooming, before moving task to next. Required before task can be prioritized.

**Effort Sizes**:
- **XS**: Extra Small (< 1 day, or units 1)
- **S**: Small (1-2 days, or units 2)
- **M**: Medium (3-5 days, or units 3)
- **L**: Large (1-2 weeks, or units 5)
- **XL**: Extra Large (> 2 weeks, or units 8)

**Note**: These are relative effort estimates, not time estimates. They reflect complexity, uncertainty, and work required.

**Steps**:

1. **Review Task Scope** ⚠️ **MUST**
   - Read task Implementation Details
   - Review Acceptance Criteria
   - Consider dependencies and complexity
   - Why MUST: Informed estimate requires understanding scope

2. **Estimate Effort** ✓ Flexible
   - Consider:
     - Technical complexity
     - Unknowns and uncertainty
     - Scope of change
     - Testing requirements
     - Integration complexity
   - Discuss with implementer if available
   - Skip reason example: "Obvious effort level from scope"

3. **Assign Effort Size** ⚠️ **MUST**
   - Choose: XS, S, M, L, or XL
   - Default to M if very uncertain
   - Why MUST: Task cannot move to next without effort estimate

4. **Update Task Frontmatter** ⚠️ **MUST**
   - Set `effort_estimate: "M"` (or appropriate size)
   - Why MUST: Single source of truth

5. **Log Event** ⚠️ **MUST**
   - Add to ledger: `{"ts":"ISO8601","user":"user_id","event":"effort_estimated","value":"M","reason":"moderate complexity, well-defined scope"}`
   - Why MUST: Audit trail of estimates

6. **Consider Split** ✓ Flexible (if L or XL)
   - If effort is L or XL, consider if task should be split
   - Large tasks often hide multiple independent sub-goals
   - Execute WF-SPLIT if appropriate
   - Skip reason example: "Large but cohesive, should not split"

**Effort Estimation Guidelines**:

| Size | Typical Scope | Examples |
|------|---------------|----------|
| XS | Tiny change, obvious implementation | "Fix typo", "Update config value", "Add log line" |
| S | Small, well-defined change | "Add validation field", "Update error message", "Simple bug fix" |
| M | Standard feature or refactor | "Add new API endpoint", "Refactor module", "Implement form" |
| L | Complex feature or significant change | "Build complete auth flow", "Major refactor", "New service integration" |
| XL | Very large, should consider splitting | "Entire subsystem", "Multiple related features" - **consider WF-SPLIT** |

**Relative Sizing**:

Effort estimates are relative to each other:
- XS is about 1/8 the effort of L
- S is about 1/4 the effort of L  
- M is about 1/2 the effort of L
- XL is about 1.5-2x the effort of L (but should be split)

**Skip Guidance**:
- **MUST steps**: Estimation and logging are required
- **Flexible steps**: Estimation discussion and split consideration adapt to context
- Be conservative: when uncertain, estimate higher

**Required Ledger Event**:
```json
{"ts":"2025-10-18T10:30:00Z","user":"@alice","event":"effort_estimated","value":"M","reason":"moderate complexity with clear acceptance criteria"}
```

**Note on Re-estimation**:

If during implementation, effort proves significantly different than estimated:
- Log a note in ledger: `{"ts":"ISO8601","user":"user_id","event":"effort_variance","original":"M","actual":"L","reason":"unexpected complexity in X"}`
- This helps calibrate future estimates
- Consider if task should be split if it's much larger than expected
