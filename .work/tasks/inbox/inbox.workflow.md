---
type: workflow
stage: inbox
parent: ../tasks.workflow.md
revisit_after_executions: 25
last_revisited: 2025-10-18
criticality: flexible
---

# Inbox Workflows

This file contains workflows specific to the inbox stage.

## WF1 - Create Task Package

**Criticality**: flexible (can adapt format/structure as needed, all adaptations are signals)

**Purpose**: Create a new task with all required files in the inbox folder.

**Steps**:

0. **Reserve Task ID** ⚠️ **MUST**
   - Read current value from `.work/tasks/ID_COUNTER.txt`
   - Use this value as the task ID (e.g., `0042`)
   - Increment the value in `ID_COUNTER.txt` and save
   - Log reservation: `{"ts":"ISO8601","user":"user_id","event":"id_reserved","desc":"Reserved ID {id}"}`
   - Why MUST: Ensures ID uniqueness and prevents collisions

1. **Create Task Folder** ⚠️ **MUST**
   - Create folder with schema `TASK-{id}-{name}` in inbox
   - Use the reserved ID from step 0
   - Why MUST: Prevents file conflicts, maintains naming consistency

2. **Create Ledger File** ⚠️ **MUST**
   - Create `{id}.ledger.jsonl` with initial events:
     - First: `{"ts":"ISO8601","user":"user_id","event":"id_reserved","desc":"Reserved ID {id}"}`
     - Second: `{"ts":"ISO8601","user":"user_id","event":"created","desc":"Task package created"}`
   - Why MUST: Ensures complete traceability from creation

3. **Create Context File** ✓ Flexible
   - Create `{id}.context.md` with hermetic context
   - Include all information needed to understand and implement task
   - Skip reason example: "Simple task, context in task card sufficient"

4. **Create Task Card** ✓ Flexible
   - Create `{id}-{name}.task.md` with goal, inputs, implementation, outputs, acceptance criteria
   - Use template provided below
   - Skip reason example: "Using adapted format for spike/research task"

5. **Update Inbox Index** ✓ Flexible
   - Add entry to `inbox/INDEX.md`
   - Skip reason example: "Batching multiple tasks, will update together"

6. **Update Root Index** ✓ Flexible
   - Add entry to root `INDEX.md`
   - Skip reason example: "Deferring to index sync workflow"

**Skip Guidance**:
- **MUST steps**: Protect system integrity - ledger and unique naming required
- **Flexible steps**: Adapt as needed - document reasons when helpful
- **Every adaptation signals** potential for workflow improvement

**Required Ledger Event**: `{"ts":"ISO8601","user":"user_id","event":"created","desc":"Task package created"}`

### Templates

**Task Card Template** (`{id}-{name}.task.md`):

```markdown
---
task_id: "{id}"
created_by: "@user"
created_date: "YYYY-MM-DD"
schema_version: "1.0"
task_name: "Short descriptive name"
stage: "inbox"
priority: null
effort_estimate: null
tags: []
current_dri: "@user"
status: "active"
blocked_by: null
paused_reason: null
depends_on: []
blocks: []
related_to: []
split_from: null
split_into: []
---

## Goal
[What needs to be accomplished]

## Inputs Required
- [Dependency 1]
- [Information needed]

## Implementation Details
[How to accomplish the goal]

## Expected Outputs
- [Deliverable 1]
- [Artifact 2]

## Acceptance Criteria
- [ ] [Testable condition 1]
- [ ] [Testable condition 2]
```

**Context File Template** (`{id}.context.md`):

```markdown
# Context: [Task Name]

## Background
[Why this task exists]

## Key Information
[Essential details needed]

## References
- [Link to related docs]
- [Link to dependencies]
```

**Initial Ledger Entry** (`{id}.ledger.jsonl`):

```json
{"ts":"2025-10-18T10:30:00Z","user":"user_id","event":"created","desc":"Task package created"}
```

**Index Entry Template**:

```markdown
| {id} | {Short Name} | @{user} | inbox | {TASK-XXX or -} |
```

---

## WF2 - Verify Task Package and Move to Backlog

**Criticality**: flexible (can adapt verification depth, all skips are signals)

**Purpose**: Verify task package completeness and correctness before moving to backlog.

**Steps**:

1. **Verify Package Completeness** ✓ Flexible
   - Check task package is complete per WF1
   - Verify ledger has creation event
   - Review context is sufficient
   - Log: `{"ts":"ISO8601","user":"user_id","event":"verified","desc":"Task verified and moved to backlog"}`
   - Skip reason example: "DRI self-verified during creation"

2. **Move to Backlog** ⚠️ **MUST**
   - Move task folder from `inbox/` to `backlog/`
   - Why MUST: Prevents task orphaning

3. **Update Indexes** ⚠️ **MUST**
   - Cut entry from `inbox/INDEX.md`
   - Paste entry into `backlog/INDEX.md`
   - Update root `INDEX.md` with new status
   - Why MUST: Maintains index consistency, prevents lost tasks

**Verification Checklist** (Flexible - adapt as needed):
- [ ] All three required files present (`{id}-{name}.task.md`, `{id}.context.md`, `{id}.ledger.jsonl`)
- [ ] Task card has all required sections (Goal, Inputs, Implementation, Outputs, Acceptance Criteria)
- [ ] Context is hermetic and self-contained
- [ ] Acceptance criteria are testable
- [ ] Ledger contains creation event
- [ ] Index entry exists in inbox/INDEX.md
- [ ] Index entry exists in root INDEX.md

**Skip Guidance**:
- Can skip verification if confident - document reason helps others
- Moving task and updating indexes are MUST - protects system integrity

**Required Ledger Event**: `{"ts":"ISO8601","user":"user_id","event":"verified","desc":"Task verified and moved to backlog"}`
