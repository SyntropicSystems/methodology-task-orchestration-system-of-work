---
type: workflow
parent: ../tasks.workflow.md
revisit_after_executions: 25
last_revisited: 2025-10-18
criticality: must
---

# Cross-Cutting Validation

## WF-VALIDATE-MOVE (Manual Pre-Flight)

**Criticality**: must

**Purpose**: Gate every move between stages with a human-runnable checklist to ensure task readiness and dependency satisfaction.

**When**: ⚠️ **MUST** run before executing WF3 (backlog → next) and WF4 (next → current)

**Inputs**: 
- `task_id`: Task identifier being moved
- `from_stage`: Current stage
- `to_stage`: Destination stage
- `DRI`: Directly Responsible Individual

**Preconditions**:
- Task package present with 3 files (`{id}-{name}.task.md`, `{id}.context.md`, `{id}.ledger.jsonl`)
- Task card frontmatter includes: `task_id`, `stage`, `priority`, `depends_on` (may be empty array)

**Steps**:

1. **Package Sanity** ⚠️ **MUST**
   - Confirm the 3 required files exist in task folder
   - Confirm task card frontmatter fields exist and `task_id` matches folder/filename
   - Why MUST: Ensures package integrity before move

2. **Index Sanity** ⚠️ **MUST**
   - Confirm one row exists in the **source** stage `INDEX.md`
   - Confirm exactly one row exists in **root** `INDEX.md`
   - (Destination indexes will be updated in the move workflow; this step only verifies source/current consistency)
   - Why MUST: Prevents orphaned or phantom tasks

3. **Dependency Validation** ⚠️ **MUST**
   - Read `depends_on` array from task frontmatter
   - For each dependency ID, verify task exists in system
   - Run **WF-DETECT-CYCLES** (see `graph-operations.workflow.md`) if dependencies were recently added or changed
   - Apply stage-specific rules:

   **For Move to `next` (WF3)**:
   - Check each dependency has a DRI and is not explicitly canceled
   - If any dependency is not yet completed:
     - **Option A**: Mark this task as `blocked` with `blocked_by: "task:TASK-XXXX"`
     - Log: `{"ts":"ISO8601","user":"{DRI}","event":"waiting_on","desc":"Waiting on TASK-XXXX"}`
     - Set flag in index: `blocked(TASK-XXXX)`
     - **Option B**: Do not move; return to backlog for re-grooming
   - If all dependencies have DRIs and are progressing, move is allowed

   **For Move to `current` (WF4)**:
   - **Default rule**: All dependencies must be in `integration`, `learning`, or `archived` stage
   - If dependency not completed:
     - **Option A**: HALT move, set task to `blocked`
     - **Option B (Exception)**: DRI records explicit waiver for concurrent work:
       ```json
       {"ts":"ISO8601","user":"{DRI}","event":"concurrent_ok","desc":"Starting in parallel with TASK-XXXX (risk accepted)"}
       ```
       - Set flag: `blocked(TASK-XXXX)` until dependency completes
       - Document rationale in context file
   - Why MUST: Enforces dependency ordering

4. **Priority Overrides** ⚠️ **MUST** (if P0)
   - If `priority: P0`, apply **P0 Fast-Track Rules** from `priority-effort.workflow.md`
   - P0 can bypass `next` stage: inbox → backlog → current
   - Apply displacement rules if current stage is full
   - Log fast-track event
   - Why MUST: Critical work needs expedited path

5. **Record Validation Result** ⚠️ **MUST**
   - **Success**:
     ```json
     {"ts":"ISO8601","user":"{DRI}","event":"validated","desc":"WF-VALIDATE-MOVE passed: {from_stage} → {to_stage}"}
     ```
   - **Failure** (do not move):
     ```json
     {"ts":"ISO8601","user":"{DRI}","event":"validation_failed","desc":"[specific reason]"}
     ```
   - Why MUST: Audit trail of validation decisions

**Outputs**:
- `validated` or `validation_failed` event in task ledger
- Clear decision: proceed with move or stop

**Validation Checklist**:
- [ ] Task package complete (3 files present)
- [ ] Frontmatter fields valid
- [ ] Task appears in source stage index
- [ ] Task appears in root index
- [ ] All dependencies exist in system
- [ ] No circular dependencies (if deps recently changed)
- [ ] Dependencies meet stage-specific rules
- [ ] P0 rules applied if applicable
- [ ] Validation event logged

**On Failure**: DO NOT proceed with task movement. Address the validation issues, log the failure reason, then retry validation.

**Required Ledger Events**:
- Success: `{"ts":"ISO8601","user":"user_id","event":"validated","desc":"WF-VALIDATE-MOVE passed: backlog → next"}`
- Failure: `{"ts":"ISO8601","user":"user_id","event":"validation_failed","desc":"Dependency TASK-0042 not completed"}`
