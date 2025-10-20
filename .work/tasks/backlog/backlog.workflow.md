---
type: workflow
stage: backlog
parent: ../tasks.workflow.md
revisit_after_executions: 25
last_revisited: 2025-10-18
criticality: flexible
---

# Backlog Workflows

This file contains workflows specific to the backlog stage.

## WF3 - Prioritization and Moving Tasks to Next

**Criticality**: flexible (can adapt prioritization approach, all adaptations are signals)

**Purpose**: Prioritize verified tasks and move them to the next queue for active work.

**Steps**:

1. **Assess Priorities** ✓ Flexible
   - Review all tasks in backlog
   - Determine next priority based on dependencies, effort, resources
   - Skip reason example: "Single task in backlog, priority obvious"

2. **Assign Priority** ✓ Flexible
   - Assign priority number within dependency graph
   - Group by effort (related tasks can run in parallel)
   - Tasks can be: effort starters, standalone, or follow-ups
   - Ensure clear relationships and ordering
   - Skip reason example: "Using different prioritization method for this effort"

3. **Validate Move (WF-VALIDATE-MOVE)** ⚠️ **MUST**
   - Execute **WF-VALIDATE-MOVE** workflow (see `workflows/validation.workflow.md`)
   - Inputs: task_id, from_stage=backlog, to_stage=next, DRI
   - This validates:
     - Package sanity (3 files present, frontmatter valid)
     - Index sanity (task in source indexes)
     - Dependency validation (dependencies exist, have DRIs, not canceled)
     - Circular dependency check (if dependencies changed)
   - **If validation fails**:
     - Task is marked as `blocked` if dependencies not ready
     - Move is aborted
     - Log `validation_failed` event
     - Address issues before retrying
   - **If validation passes**:
     - Log `validated` event
     - Proceed with move
   - Why MUST: Enforces dependency rules and prevents invalid moves

4. **Log Prioritization** ⚠️ **MUST**
   - Add event to task ledger: `{"ts":"ISO8601","user":"user_id","event":"prioritized","desc":"Task prioritized and moved to next"}`
   - Why MUST: Maintains complete event history

5. **Move to Next** ⚠️ **MUST**
   - Move task folder from `backlog/` to `next/`
   - Update indexes:
     - Cut entry from `backlog/INDEX.md`
     - Paste entry into `next/INDEX.md` in correct group and priority order
     - Update root `INDEX.md` with new status
   - Number task in index with its order within effort group
   - Ensure numbers are consecutive and monotonic within group
   - Why MUST: Prevents task orphaning, maintains index consistency

**Prioritization Considerations**:
- Dependencies between tasks
- Effort grouping (related tasks)
- Resource availability
- Strategic importance
- Blocking vs non-blocking work
- Parallel work streams

**Skip Guidance**:
- **MUST steps**: Logging and index updates protect system integrity
- **Flexible steps**: Adapt prioritization approach as needed
- **Every adaptation signals** potential for improved prioritization methods

**Required Ledger Event**: `{"ts":"ISO8601","user":"user_id","event":"prioritized","desc":"Task prioritized and moved to next"}`
