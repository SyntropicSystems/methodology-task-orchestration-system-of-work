---
type: workflow
stage: system
parent: ../tasks.workflow.md
revisit_after_executions: 25
last_revisited: 2025-10-18
criticality: flexible
---

# State Management Workflows

This file contains workflows for managing task state transitions (blocked, paused, canceled) and cascade operations.

## WF-BLOCK - Mark Task as Blocked

**Criticality**: flexible (can adapt documentation)

**Purpose**: Mark a task as unable to proceed due to dependency or external factor.

**When**: Task cannot continue because:
- Waiting on another task to complete
- External dependency (API access, stakeholder decision, etc.)
- Missing information
- Resource unavailable

**Steps**:

1. **Identify Blocker** ✓ Flexible
   - Determine what's blocking the task
   - Categorize: task dependency, external, decision, or information
   - Skip reason example: "Blocker obvious, documented in ledger only"

2. **Update Task Frontmatter** ⚠️ **MUST**
   - Set `status: blocked`
   - Set `blocked_by` field with specific reason:
     - `"task:TASK-0042"` - Blocked by another task
     - `"external:API access approval"` - External dependency
     - `"decision:Architecture review pending"` - Decision needed
     - `"info:Requirements clarification from stakeholder"` - Information needed
   - Why MUST: Single source of truth for task state

3. **Update Index Flags** ⚠️ **MUST**
   - Update stage INDEX.md: add flag `blocked(TASK-0042)` or `blocked(external:desc)`
   - Update root INDEX.md: same flag
   - Why MUST: Makes blocked tasks visible at a glance

4. **Log Event** ⚠️ **MUST**
   - Add to ledger: `{"ts":"ISO8601","user":"user_id","event":"blocked","reason":"waiting on TASK-0042","blocked_by":"TASK-0042"}`
   - Why MUST: Audit trail of state changes

5. **Create Follow-up Task** ✓ Flexible (as needed)
   - If blocker requires action, create a task to track blocker resolution
   - Link tasks via `related_to` field
   - Skip reason example: "Blocker is external, no actionable task to create"

6. **Notify Stakeholders** ✓ Flexible
   - Inform DRI or stakeholders that task is blocked
   - Skip reason example: "DRI is blocker owner, already aware"

**Blocker Type Examples**:
- `"task:TASK-0042"` - Depends on another task
- `"external:API access approval from security team"` - External process
- `"decision:Choose between Postgres vs MySQL"` - Decision point
- `"info:Clarify acceptance criteria with product owner"` - Information gap

**Skip Guidance**:
- **MUST steps**: State updates and logging protect integrity
- **Flexible steps**: Communication and follow-ups adapt to context

**Required Ledger Event**:
```json
{"ts":"2025-10-18T10:30:00Z","user":"@alice","event":"blocked","reason":"waiting on TASK-0042 to be integrated","blocked_by":"task:TASK-0042"}
```

---

## WF-UNBLOCK - Resume Blocked Task

**Criticality**: flexible

**Purpose**: Mark a blocked task as ready to proceed after blocker is resolved.

**When**: The blocking condition has been resolved.

**Steps**:

1. **Verify Blocker Resolved** ⚠️ **MUST**
   - Confirm the blocker is actually resolved
   - Check task status if blocked by another task
   - Verify external dependency if external block
   - Why MUST: Don't unblock prematurely

2. **Update Task Frontmatter** ⚠️ **MUST**
   - Set `status: active`
   - Clear `blocked_by` field (set to `null`)
   - Why MUST: Single source of truth

3. **Update Index Flags** ⚠️ **MUST**
   - Remove `blocked(...)` flag from stage INDEX.md
   - Remove `blocked(...)` flag from root INDEX.md
   - Optionally add `dep_ready` flag if all dependencies satisfied
   - Why MUST: Keeps indexes current

4. **Log Event** ⚠️ **MUST**
   - Add to ledger: `{"ts":"ISO8601","user":"user_id","event":"unblocked","reason":"TASK-0042 completed and integrated","previously_blocked_by":"task:TASK-0042"}`
   - Why MUST: Complete provenance

5. **Notify DRI** ✓ Flexible
   - Alert DRI that task is ready to proceed
   - Skip reason example: "DRI monitoring, will see status change"

6. **Re-prioritize** ✓ Flexible (if needed)
   - If task sat blocked for a while, consider re-evaluating priority
   - Skip reason example: "Priority unchanged"

**Skip Guidance**:
- **MUST steps**: Verification and state updates prevent premature unblocking
- **Flexible steps**: Communication adapts to team process

**Required Ledger Event**:
```json
{"ts":"2025-10-18T10:30:00Z","user":"@alice","event":"unblocked","reason":"TASK-0042 completed and integrated","previously_blocked_by":"task:TASK-0042"}
```

---

## WF-PAUSE - Temporarily Suspend Task

**Criticality**: flexible

**Purpose**: Temporarily suspend a task with intention to resume later.

**When**:
- DRI unavailable (vacation, leave)
- Higher priority work emerged
- Waiting for stakeholder input
- Resource constraints
- Strategic pivot (but not canceling)

**Steps**:

1. **Document Pause Reason** ⚠️ **MUST**
   - Record clear reason for pause
   - Set expected resume date if known
   - Why MUST: Future context for resumption

2. **Update Task Frontmatter** ⚠️ **MUST**
   - Set `status: paused`
   - Set `paused_reason` with explanation
   - Set `expected_resume_date` (optional but recommended)
   - Set `paused_date` to current date
   - Why MUST: Single source of truth

3. **Document Current State** ⚠️ **MUST**
   - Update context file with current progress/state
   - Note what's completed, what remains
   - Document any work-in-progress state
   - Why MUST: Enables smooth resumption

4. **Move Task** ⚠️ **MUST** (stage-dependent)
   - If in `current/` → Move to `next/` (active work stage is for active tasks only)
   - If in `next/` → Keep in `next/`, just mark paused
   - If in `backlog/` → Keep in `backlog/`
   - If in other stages → Pause only applies to backlog/next/current
   - Update all indexes (stage and root)
   - Why MUST: Paused tasks exit active work

5. **Update Index Flags** ⚠️ **MUST**
   - Add flag `paused(YYYY-MM-DD)` with resume date
   - Update both stage and root indexes
   - Why MUST: Makes paused tasks visible

6. **Log Event** ⚠️ **MUST**
   - Add to ledger: `{"ts":"ISO8601","user":"user_id","event":"paused","reason":"DRI on vacation until 2025-11-01","expected_resume":"2025-11-01","paused_from_stage":"current"}`
   - Why MUST: Complete history

7. **Create Checkpoint** ✓ Flexible
   - Save work-in-progress if applicable
   - Commit code, save documents, etc.
   - Skip reason example: "Nothing to checkpoint, task just started"

**Important**: While paused, task cannot move forward to more advanced stages. Must execute WF-RESUME before moving forward. Backward moves (rollback) are allowed while paused.

**Pause Reason Examples**:
- `"vacation: DRI unavailable until 2025-11-01"`
- `"priorities: P0 incident requires immediate attention"`
- `"information: Waiting for stakeholder review by 2025-11-15"`
- `"dependencies: Prerequisites not yet ready"`
- `"resources: Budget approval pending"`

**Skip Guidance**:
- **MUST steps**: State documentation and task movement maintain system integrity
- **Flexible steps**: Checkpointing adapts to work type

**Required Ledger Event**:
```json
{"ts":"2025-10-18T10:30:00Z","user":"@alice","event":"paused","reason":"DRI on vacation until 2025-11-01","expected_resume":"2025-11-01","paused_from_stage":"current"}
```

---

## WF-RESUME - Restart Paused Task

**Criticality**: flexible

**Purpose**: Restart a previously paused task.

**When**: Pause reason is resolved and ready to continue work.

**Steps**:

1. **Verify Pause Reason Resolved** ⚠️ **MUST**
   - Confirm the reason for pause no longer applies
   - Check expected resume date has passed
   - Why MUST: Don't resume prematurely

2. **Review Context** ✓ Flexible
   - Re-read context file to understand current state
   - Review what was completed before pause
   - Check if acceptance criteria changed
   - Skip reason example: "Short pause, context fresh in mind"

3. **Update Task Frontmatter** ⚠️ **MUST**
   - Set `status: active`
   - Clear `paused_reason` (set to `null`)
   - Clear `expected_resume_date`
   - Why MUST: Single source of truth

4. **Move Task** ⚠️ **MUST** (if resuming active work)
   - If resuming active work, move from `next/` to `current/`
   - If remaining in planning, keep in `next/` or `backlog/`
   - Update all indexes (stage and root)
   - Why MUST: Current reflects active work

5. **Update Index Flags** ⚠️ **MUST**
   - Remove `paused(...)` flag from stage INDEX.md
   - Remove `paused(...)` flag from root INDEX.md
   - Why MUST: Keeps indexes current

6. **Log Event** ⚠️ **MUST**
   - Add to ledger: `{"ts":"ISO8601","user":"user_id","event":"resumed","reason":"DRI returned from vacation","paused_duration_days":14,"resumed_to_stage":"current"}`
   - Calculate pause duration for metrics
   - Why MUST: Complete history

7. **Re-verify Prerequisites** ✓ Flexible
   - Check if dependencies or context changed during pause
   - Update implementation plan if needed
   - Skip reason example: "Short pause, nothing changed"

**Skip Guidance**:
- **MUST steps**: Verification and state updates ensure clean resumption
- **Flexible steps**: Context review adapts to pause duration

**Required Ledger Event**:
```json
{"ts":"2025-10-18T12:00:00Z","user":"@alice","event":"resumed","reason":"DRI returned from vacation","paused_duration_days":14,"resumed_to_stage":"current"}
```

---

## WF-CANCEL - Abandon Task

**Criticality**: flexible

**Purpose**: Mark a task as abandoned - will not be completed.

**When**:
- Requirements changed, work no longer needed
- Feature superseded by different solution
- Business need eliminated
- Technically infeasible
- Duplicate of another task

**Important**: Canceled is a **terminal state**. Task cannot be uncanceled. If work is revived, it's a new task.

**Steps**:

1. **Document Cancellation Reason** ⚠️ **MUST**
   - Record detailed explanation of why canceled
   - This is permanent record, be thorough
   - Why MUST: Historical context for future reference

2. **Document What Was Completed** ⚠️ **MUST** (if any work done)
   - Note any partial work completed
   - Document percentage completion
   - List any artifacts created
   - Why MUST: Captures investment made

3. **Check for Dependent Tasks** ⚠️ **MUST**
   - Run **WF-CASCADE-CANCEL** (see below) to handle tasks that depend on this one
   - Why MUST: Prevents orphaned dependencies

4. **Create Follow-up Tasks** ✓ Flexible (if needed)
   - If cancellation creates new work, create new tasks
   - Example: "Clean up prototype code from canceled feature"
   - Link via `related_to` field
   - Skip reason example: "No cleanup needed"

5. **Update Task Frontmatter** ⚠️ **MUST**
   - Set `status: canceled`
   - Set `canceled_reason` with detailed explanation
   - Why MUST: Single source of truth

6. **Log Follow-ups** ⚠️ **MUST** (if created)
   - Log each follow-up task: `{"ts":"ISO8601","user":"user_id","event":"followup_created","followup_task":"TASK-0099","reason":"cleanup work"}`
   - Why MUST: Maintains complete graph

7. **Move to Archived** ⚠️ **MUST**
   - Move task folder from current stage to `archived/`
   - Update all indexes (remove from stage, update root to archived with canceled status)
   - Why MUST: Canceled tasks are archived for history

8. **Log Cancellation** ⚠️ **MUST**
   - Add to ledger: `{"ts":"ISO8601","user":"user_id","event":"canceled","reason":"requirements changed, feature no longer needed","from_stage":"next","completed_percentage":30}`
   - Why MUST: Complete provenance

**Cancellation Reason Examples**:
- `"requirements_changed: Stakeholder changed direction to mobile-first approach"`
- `"superseded: Feature covered by new third-party integration"`
- `"no_longer_needed: Business pivot eliminated this need"`
- `"infeasible: Technical constraints make this approach non-viable"`
- `"duplicate: Realized this duplicates TASK-0123"`

**Skip Guidance**:
- **MUST steps**: Documentation and archiving preserve history
- **Flexible steps**: Follow-up creation adapts to context
- Even canceled tasks provide learning value when well-documented

**Required Ledger Events**:
- Cancellation: `{"ts":"ISO8601","user":"user_id","event":"canceled","reason":"[detailed reason]","from_stage":"next","completed_percentage":30}`
- Follow-ups (if any): `{"ts":"ISO8601","user":"user_id","event":"followup_created","followup_task":"TASK-0099","reason":"cleanup work"}`

---

## WF-CASCADE-CANCEL - Handle Dependent Tasks When Canceling

**Criticality**: must

**Purpose**: When canceling a task, ensure dependent tasks are properly handled to prevent broken dependencies.

**When**: Triggered automatically as part of WF-CANCEL when the canceled task has dependents.

**Steps**:

1. **Identify Dependents** ⚠️ **MUST**
   - Search all stage indexes for tasks listing this task in their `depends_on` or referencing in `Dependencies` column
   - Search all task frontmatter for `depends_on` arrays containing this task ID
   - Why MUST: Must handle all affected tasks

2. **For Each Dependent Task** ⚠️ **MUST**
   
   **If dependent is in `inbox`, `backlog`, or `next`:**
   - Mark task as `blocked` with `blocked_by: "task:TASK-XXXX (canceled)"`
   - Set flag: `blocked(TASK-XXXX:canceled)`
   - Log in dependent's ledger:
     ```json
     {"ts":"ISO8601","user":"{DRI}","event":"cascade_blocked_by_cancel","desc":"Upstream TASK-XXXX was canceled","blocker":"TASK-XXXX"}
     ```
   - DRI must decide action:
     - **Retarget**: Find alternate dependency via **WF-LINK** (see `graph-operations.workflow.md`)
     - **Split**: Break into parts where some don't need canceled dependency via **WF-SPLIT**
     - **Cancel**: Cancel this dependent too if it can't proceed without the dependency
   
   **If dependent is in `current` or `integration`:**
   - DRI decides immediately:
     - **Pause**: Execute WF-PAUSE to suspend work while retargeting
       ```json
       {"ts":"ISO8601","user":"{DRI}","event":"paused","reason":"Upstream TASK-XXXX canceled, re-planning needed"}
       ```
     - **Retarget**: Update `depends_on` via **WF-LINK**, document workaround in context
     - **Cancel**: If work can't proceed, execute WF-CANCEL on dependent
   
   Why MUST: Broken dependencies must be resolved
   
3. **Log Cascade in Upstream Task** ⚠️ **MUST**
   - In the canceled task's ledger, log:
     ```json
     {"ts":"ISO8601","user":"{DRI}","event":"cascade_cancel","desc":"Affected dependents: TASK-AAAA, TASK-BBBB","dependents":["TASK-AAAA","TASK-BBBB"]}
     ```
   - Why MUST: Complete record of cascade impact

4. **Notify Affected DRIs** ⚠️ **MUST**
   - Alert all affected task DRIs that their tasks are blocked by cancellation
   - Provide context on why upstream was canceled
   - Why MUST: DRIs need to take action on their tasks

**Required Ledger Events**:
- Upstream (canceled task): `{"ts":"ISO8601","user":"user_id","event":"cascade_cancel","desc":"Affected dependents: TASK-AAAA, TASK-BBBB","dependents":["TASK-AAAA","TASK-BBBB"]}`
- Each dependent: `{"ts":"ISO8601","user":"user_id","event":"cascade_blocked_by_cancel","desc":"Upstream TASK-XXXX was canceled","blocker":"TASK-XXXX"}`

---

## WF-CASCADE-PAUSE - Handle Dependent Tasks When Pausing

**Criticality**: flexible

**Purpose**: When pausing a task, optionally propagate pause to dependent tasks if they cannot proceed without it.

**When**: Triggered optionally as part of WF-PAUSE if the paused task has dependents that will be blocked.

**Steps**:

1. **Identify Dependents** ✓ Flexible
   - Search for tasks that depend on this task (same method as WF-CASCADE-CANCEL)
   - Assess if dependents can proceed without this task completing
   - Skip reason example: "Dependents can continue in parallel"

2. **For Each Affected Dependent** ✓ Flexible
   
   **If dependent cannot proceed:**
   - Mark as `blocked` with `blocked_by: "task:TASK-XXXX (paused)"`
   - Set flag: `blocked(TASK-XXXX:paused)`
   - Log in dependent's ledger:
     ```json
     {"ts":"ISO8601","user":"{DRI}","event":"cascade_blocked_by_pause","desc":"Waiting on TASK-XXXX to resume","blocker":"TASK-XXXX"}
     ```
   - Consider alternative: Temporary re-sequencing or splitting work that can proceed independently
   
   **If dependent can proceed in parallel:**
   - No action needed
   - Optionally log informational note:
     ```json
     {"ts":"ISO8601","user":"{DRI}","event":"dependency_paused","desc":"TASK-XXXX paused but work can continue"}
     ```
   
   Skip reason example: "Dependencies are loose, work can proceed"

3. **When Upstream Resumes** ⚠️ **MUST**
   - When paused task executes WF-RESUME, check for cascade-blocked dependents
   - For each affected dependent, log `unblocked` event:
     ```json
     {"ts":"ISO8601","user":"{DRI}","event":"unblocked","reason":"TASK-XXXX resumed","previously_blocked_by":"task:TASK-XXXX"}
     ```
   - Remove `blocked(...)` flags from indexes
   - Why MUST: Clean up cascade effects

**Skip Guidance**:
- Cascade pause is optional - use judgment based on dependency tightness
- Loose dependencies (inspiration, reference) don't require cascade
- Tight dependencies (builds on, requires output) may require cascade

**Required Ledger Events** (if cascade applied):
- Each dependent: `{"ts":"ISO8601","user":"user_id","event":"cascade_blocked_by_pause","desc":"Waiting on TASK-XXXX to resume","blocker":"TASK-XXXX"}`
- On resume: `{"ts":"ISO8601","user":"user_id","event":"unblocked","reason":"TASK-XXXX resumed","previously_blocked_by":"task:TASK-XXXX"}`
