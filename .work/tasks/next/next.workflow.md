---
type: workflow
stage: next
parent: ../tasks.workflow.md
revisit_after_executions: 25
last_revisited: 2025-10-18
criticality: flexible
---

# Next Workflows

This file contains workflows specific to the next stage.

## WF4 - Move to Current

**Criticality**: flexible (can adapt selection and verification approach, all adaptations are signals)

**Purpose**: Move prioritized task from next queue to active implementation.

**Steps**:

1. **Select Task** ✓ Flexible
   - Choose highest priority task from next queue
   - Skip reason example: "Picking different task due to urgent need"

2. **Validate Move (WF-VALIDATE-MOVE)** ⚠️ **MUST**
   - Execute **WF-VALIDATE-MOVE** workflow (see `workflows/validation.workflow.md`)
   - Inputs: task_id, from_stage=next, to_stage=current, DRI
   - This validates:
     - Package sanity (3 files present, frontmatter valid)
     - Index sanity (task in source indexes)
     - Dependency validation with **strict rules for current**:
       - Default: All dependencies must be in `integration`, `learning`, or `archived` stage
       - Exception: DRI can log `concurrent_ok` waiver with documented risk
     - No circular dependencies
   - **If validation fails**:
     - Task is marked as `blocked` if dependencies not complete
     - Move is aborted
     - Log `validation_failed` event
     - Address issues before retrying
   - **If validation passes**:
     - Log `validated` event
     - Proceed with move
   - Why MUST: Enforces stricter dependency rules for active work

3. **Verify Prerequisites** ✓ Flexible
   - DRI is available and commits to implementation
   - Required resources/information available
   - Skip reason example: "Prerequisites clear from validation step"

4. **Move Task Folder** ⚠️ **MUST**
   - Physically move task folder from `next/` to `current/`
   - Why MUST: Prevents task orphaning

4. **Update Indexes** ⚠️ **MUST**
   - Cut task entry from `next/INDEX.md`
   - Paste task entry into `current/INDEX.md`
   - Update root `INDEX.md` with new status
   - Why MUST: Maintains index consistency

5. **Log Event** ⚠️ **MUST**
   - Add to task ledger: `{"ts":"ISO8601","user":"user_id","event":"started","desc":"Task moved to current and implementation started"}`
   - Why MUST: Tracks task state transitions

6. **Create Implementation Plan** ✓ Flexible
   - For complex tasks, create implementation plan in task context
   - Skip reason example: "Simple task, plan not needed"

**Skip Guidance**:
- **MUST steps**: Task movement, index updates, and logging protect system integrity
- **Flexible steps**: Adapt selection and planning as needed
- **Every adaptation signals** potential for workflow improvement

**Verification Checklist**:
- [ ] Task folder moved to `current/`
- [ ] All three INDEX.md files updated
- [ ] Ledger event logged
- [ ] DRI committed

**Note**: Future-wise we might split tasks in sub-tasks, which are contained in a package but have sub-packages, essentially where for example documentation can be an independent step. But for now we keep it simple until data says otherwise.
