---
type: workflow
stage: system
parent: ../tasks.workflow.md
revisit_after_executions: 25
last_revisited: 2025-10-18
criticality: flexible
---

# Graph Operations Workflows

This file contains workflows for managing the task dependency graph (links, splits, merges, cycle detection).

## WF-SPLIT - Break Task into Multiple Tasks

**Criticality**: flexible

**Purpose**: Break one large task into multiple smaller, focused tasks.

**When**:
- Task is too large (> estimated threshold, e.g., > L effort)
- Task reveals multiple independent sub-goals
- Different parts require different skills
- Parallel work is possible
- Clearer accountability desired

**Important**: **Single-level only**. No nested hierarchies (Epic→Story→Task). Keep it flat.

**Default Rule**: **All children inherit all parent's dependencies** unless explicitly documented otherwise.

**Steps**:

1. **Plan the Split** ⚠️ **MUST**
   - Identify how to break task into logical pieces
   - Ensure each new task is focused and independently valuable
   - Determine dependencies between split tasks
   - Why MUST: Poor split creates confusion

2. **Create New Task Packages** ⚠️ **MUST**
   - For each split task, execute WF1 to create new task in `inbox/`
   - Reserve new task IDs from ID_COUNTER.txt
   - Copy relevant portions of original context to each new task
   - Distribute acceptance criteria across new tasks
   - Why MUST: Proper task creation ensures all follow normal lifecycle

3. **Link Split Tasks** ⚠️ **MUST**
   - In original task frontmatter:
     ```yaml
     split_into: ["TASK-0125", "TASK-0126", "TASK-0127"]
     split_date: "2025-10-18"
     split_reason: "Task too large, broke into 3 focused tasks"
     ```
   - In each new task frontmatter:
     ```yaml
     split_from: "TASK-0042"
     split_part: "1 of 3"  # or "2 of 3", "3 of 3"
     ```
   - Why MUST: Maintains provenance chain

4. **Inherit and Distribute Dependencies** ⚠️ **MUST**
   
   **Default Inheritance Rule**:
   - **Parent `depends_on`** → All children inherit by default
   - Rationale: If parent needed X completed first, children likely do too
   
   **Parent `blocks` (Inverse Dependencies)**:
   - User must decide distribution strategy:
     - **Option A**: One child blocks the dependent task (most common)
     - **Option B**: All children must complete before dependent unblocks
     - **Option C**: Create aggregator task that waits for all children
   - Document decision in split_reason
   
   **Deviations from Default**:
   - If a child should NOT inherit a parent dependency, log in child immediately after creation:
     ```json
     {"ts":"ISO8601","user":"{DRI}","event":"deps_updated","desc":"Removed TASK-XXXX (not applicable to this split part)","reason":"[explanation]"}
     ```
   - If a child needs DIFFERENT dependencies, log:
     ```json
     {"ts":"ISO8601","user":"{DRI}","event":"deps_updated","desc":"Replaced TASK-XXXX with TASK-YYYY","reason":"[explanation]"}
     ```
   
   **Between Children**:
   - If children have sequential dependencies (Child B depends on Child A), set via `depends_on` in Child B's frontmatter
   
   Why MUST: Maintains graph integrity after split

5. **Log in Original Task** ⚠️ **MUST**
   - Add to original task ledger: 
     ```json
     {"ts":"ISO8601","user":"user_id","event":"split","into":["TASK-0125","TASK-0126","TASK-0127"],"reason":"task too large, estimated XL effort"}
     ```
   - Why MUST: Provenance of split decision

6. **Log in New Tasks** ⚠️ **MUST**
   - In each new task ledger: 
     ```json
     {"ts":"ISO8601","user":"user_id","event":"created_from_split","parent":"TASK-0042","part":"1 of 3"}
     ```
   - Why MUST: Provenance from child perspective

7. **Move Original to Archived** ⚠️ **MUST**
   - Set original task `status: split`
   - Move to `archived/` with flag `split(children)`
   - Update all indexes
   - Why MUST: Original task is superseded by children

8. **Assign DRIs** ✓ Flexible
   - Assign same or different DRIs to split tasks
   - Consider skill match for each piece
   - Skip reason example: "Same DRI for all split tasks"

**Split Decision Criteria**:
- Task estimated > L or XL effort
- Multiple independent goals identified
- Different expertise needed for different parts
- Opportunity for parallel work
- Accountability would be clearer with separation

**Dependency Inheritance Examples**:

**Example 1: Simple Inheritance (Default)**
```
Before Split:
  TASK-0042 "Build authentication system"
    depends_on: [TASK-0030]
    blocks: [TASK-0099]

After Split:
  TASK-0042 → archived (status: split)
  
  TASK-0125 "Implement login/logout API"
    depends_on: [TASK-0030]  ← inherited
    split_from: TASK-0042
  
  TASK-0126 "Build session management"
    depends_on: [TASK-0030]  ← inherited
    split_from: TASK-0042
  
  TASK-0127 "Create password reset flow"
    depends_on: [TASK-0030]  ← inherited
    blocks: [TASK-0099]  ← inverse dep assigned to this child
    split_from: TASK-0042
```

**Example 2: Deviation from Inheritance**
```
TASK-0200 "Refactor entire codebase"
  depends_on: [TASK-0150 "Approve refactor plan"]
  
Split into:
  TASK-0201 "Refactor backend" 
    depends_on: [TASK-0150]  ← inherited
  
  TASK-0202 "Refactor frontend"
    depends_on: [TASK-0150]  ← inherited
  
  TASK-0203 "Update documentation"
    depends_on: []  ← REMOVED dependency
    Ledger: {"event":"deps_updated","desc":"Removed TASK-0150","reason":"Documentation can start independently"}
```

**Example 3: Sequential Children**
```
TASK-0300 "Implement payment flow"
  depends_on: [TASK-0280]

Split into:
  TASK-0301 "Backend payment API"
    depends_on: [TASK-0280]  ← inherited from parent
  
  TASK-0302 "Frontend payment UI"
    depends_on: [TASK-0280, TASK-0301]  ← inherited + new dependency on sibling
```

**Skip Guidance**:
- **MUST steps**: Proper task creation and linking maintains graph integrity
- **Flexible steps**: DRI assignment adapts to context
- Don't over-split: each new task should be meaningfully independent

**Required Ledger Events**:
- Original task: `{"ts":"ISO8601","user":"user_id","event":"split","into":["TASK-0125","TASK-0126","TASK-0127"],"reason":"task too large"}`
- Each new task: `{"ts":"ISO8601","user":"user_id","event":"created_from_split","parent":"TASK-0042","part":"1 of 3"}`
- Dependency deviations: `{"ts":"ISO8601","user":"user_id","event":"deps_updated","desc":"Removed/Added TASK-XXXX","reason":"[explanation]"}`

---

## WF-MERGE - Combine Duplicate Tasks

**Criticality**: flexible

**Purpose**: Combine two tasks when they are discovered to be duplicates or overlapping.

**When**:
- Discover two tasks are solving the same problem
- Tasks have significant overlap in scope
- Better to combine than coordinate separately

**Steps**:

1. **Identify Primary and Duplicate** ⚠️ **MUST**
   - Designate one task as the **primary** (TASK-A) - this survives
   - Designate the other as the **duplicate** (TASK-B) - this gets canceled
   - Consider: which has more progress, better context, clearer scope
   - Why MUST: Clear designation prevents confusion

2. **Copy Essential Information** ⚠️ **MUST**
   - From TASK-B to TASK-A:
     - Copy any unique context or implementation details
     - Copy any unique acceptance criteria
     - Copy relevant ledger events as context summary (not as events)
   - Update TASK-A's context file with merged information
   - Add section to TASK-A context file:
     ```markdown
     ## Merged From TASK-B
     Relevant context from TASK-B:
     - [Key point 1]
     - [Key point 2]
     
     Ledger highlights from TASK-B:
     - [timestamp] [event]: [description]
     - [timestamp] [event]: [description]
     
     Remaining differences: [what wasn't merged]
     ```
   - Why MUST: Preserves all valuable information

3. **Merge Dependencies** ⚠️ **MUST**
   - In TASK-A frontmatter:
     - `depends_on`: Union of TASK-A and TASK-B dependencies (deduplicate)
     - `blocks`: Union of inverse dependencies
     - `related_to`: Add TASK-B ID
   - Log:
     ```json
     {"ts":"ISO8601","user":"user_id","event":"deps_updated","desc":"Merged dependencies from TASK-B","added":["TASK-XXXX"],"reason":"merge"}
     ```
   - Why MUST: Maintains complete dependency graph

4. **Log in Primary Task** ⚠️ **MUST**
   - Add to TASK-A ledger: 
     ```json
     {"ts":"ISO8601","user":"user_id","event":"merged_from","source_task":"TASK-B","reason":"duplicate: solving same problem"}
     ```
   - Why MUST: Complete history in surviving task

5. **Execute WF-CANCEL on Duplicate** ⚠️ **MUST**
   - Run WF-CANCEL workflow on TASK-B
   - Use cancellation reason: `"duplicate: merged into TASK-A"`
   - This will:
     - Move TASK-B to archived
     - Update indexes
     - Log cancellation event
     - Trigger WF-CASCADE-CANCEL if TASK-B has dependents
   - Why MUST: Properly closes duplicate task

6. **Update Cross-References** ⚠️ **MUST** (if needed)
   - If any other tasks reference TASK-B in their `depends_on` or `related_to` fields:
     - Update them to reference TASK-A instead
     - Execute WF-LINK to update each reference
     - Log the change in each affected task's ledger
   - Why MUST: Maintains graph integrity

**Merge Decision Matrix**:

| Scenario | Action |
|----------|--------|
| Identical goals, different DRIs | Merge, choose more advanced task as primary |
| Identical goals, same DRI | Merge, user error |
| 80%+ overlap | Merge, document differences in context |
| 50-80% overlap | Consider split or keep separate with coordination |
| <50% overlap | Keep separate, use `related_to` to link |

**Skip Guidance**:
- **MUST steps**: Information preservation and proper cancellation maintain system integrity
- Consider carefully before merging - when in doubt, keep separate and coordinate

**Required Ledger Events**:
- Primary task (TASK-A): `{"ts":"ISO8601","user":"user_id","event":"merged_from","source_task":"TASK-B","reason":"duplicate"}`
- Duplicate task (TASK-B): `{"ts":"ISO8601","user":"user_id","event":"canceled","reason":"duplicate: merged into TASK-A",...}` (via WF-CANCEL)
- Dependency updates: `{"ts":"ISO8601","user":"user_id","event":"deps_updated","desc":"Merged dependencies from TASK-B",...}`

**Example**:

Before:
```
TASK-0042 "Build user authentication" (in next, 60% defined)
TASK-0089 "Implement login system" (in backlog, 80% defined)
```

After:
```
TASK-0042 "Build user authentication" (in next)
  - related_to: ["TASK-0089"]
  - depends_on: merged from both tasks
  - Context updated with TASK-0089's unique details
  - Ledger has merged_from event

TASK-0089 "Implement login system" (in archived, status: canceled)
  - canceled_reason: "duplicate: merged into TASK-0042"
```

---

## WF-LINK - Add/Remove Dependency

**Criticality**: flexible

**Purpose**: Add or remove a dependency relationship between tasks mid-flight.

**When**:
- Discover new dependency after task is already in progress
- Dependency relationship changes
- Initial dependencies were incorrect
- Task scope changes to require new dependencies

**Steps**:

1. **Identify Dependency Change** ✓ Flexible
   - Determine what dependency needs to be added or removed
   - Verify the dependency task exists (if adding)
   - Skip reason example: "Obvious from context"

2. **Run WF-DETECT-CYCLES** ⚠️ **MUST** (if adding)
   - Before adding a dependency, verify it won't create a cycle
   - Execute WF-DETECT-CYCLES workflow (see below)
   - If cycle detected, HALT and resolve cycle first
   - Why MUST: Prevents circular dependencies that would deadlock tasks

3. **Update Task Frontmatter** ⚠️ **MUST**
   - Modify `depends_on` array:
     - Adding: Add task ID to array (e.g., `depends_on: ["TASK-0037", "TASK-0042"]`)
     - Removing: Remove task ID from array
   - Why MUST: Single source of truth for dependencies

4. **Check If Should Block** ⚠️ **MUST** (when adding dependency)
   - If adding a new dependency, check status of the dependency task
   - If dependency is not yet in `integration` or `archived`:
     - Consider executing WF-BLOCK to mark task as blocked
     - Update task status to `blocked` if appropriate
     - Add flag: `blocked(TASK-XXXX)`
   - Why MUST: Prevents working on tasks with unmet dependencies

5. **Update Index Dependencies Column** ⚠️ **MUST**
   - Update the Dependencies column in stage INDEX.md
   - Update the Dependencies column in root INDEX.md
   - Why MUST: Keeps indexes accurate

6. **Update Index Flags** ⚠️ **MUST** (if blocking)
   - If task became blocked, add `blocked(TASK-XXXX)` flag
   - If dependency removed and no longer blocked, remove flag
   - Why MUST: Visual indicator of blocked state

7. **Log Event** ⚠️ **MUST**
   - Add to ledger: 
     ```json
     {"ts":"ISO8601","user":"user_id","event":"dependency_updated","action":"add|remove","task_id":"TASK-0042","reason":"discovered new prerequisite during implementation"}
     ```
   - Why MUST: Audit trail of graph changes

**Examples**:

Adding dependency:
```json
{"ts":"2025-10-18T10:30:00Z","user":"@alice","event":"dependency_updated","action":"add","task_id":"TASK-0042","reason":"discovered during implementation that auth system is required"}
```

Removing dependency:
```json
{"ts":"2025-10-18T10:30:00Z","user":"@alice","event":"dependency_updated","action":"remove","task_id":"TASK-0037","reason":"refactored to eliminate dependency"}
```

**Skip Guidance**:
- **MUST steps**: Dependency tracking and state updates maintain graph integrity
- **Flexible steps**: Can adapt communication and documentation

**Required Ledger Event**:
```json
{"ts":"ISO8601","user":"user_id","event":"dependency_updated","action":"add|remove","task_id":"TASK-XXXX","reason":"[explanation]"}
```

---

## WF-DETECT-CYCLES - Verify DAG Structure

**Criticality**: must (when adding/changing dependencies)

**Purpose**: Detect circular dependencies that would create deadlock in the task graph.

**When**: 
- Before adding a new dependency via WF-LINK
- After completing a WF-SPLIT that creates new dependencies
- Periodically as validation (optional)

**Method**: Manual Depth-First Search (DFS)

**Steps**:

1. **Initialize Tracking** ⚠️ **MUST**
   - Start at the task where dependency is being added: `{task_id}`
   - Create a "visited" set: empty `{}`
   - Create a "stack" set: empty `{}`
   - Why MUST: Need to track traversal state

2. **Perform DFS** ⚠️ **MUST**
   - For the proposed new dependency `new_dep`:
     - If `new_dep == {task_id}`: **CYCLE DETECTED** (self-loop)
     - If `new_dep` in stack: **CYCLE DETECTED**
     - Add `new_dep` to stack
     - Read `new_dep`'s frontmatter and get its `depends_on` array
     - For each dependency `d` in that array:
       - Recursively check `d` using same process
       - If recursion finds cycle, propagate detection upward
     - Remove `new_dep` from stack (backtrack)
     - Add `new_dep` to visited (don't revisit)
   - If no cycles detected in full traversal, dependency is safe
   - Why MUST: Complete graph traversal required

3. **If Cycle Detected** ⚠️ **MUST**
   - HALT the dependency addition
   - Log in task ledger:
     ```json
     {"ts":"ISO8601","user":"user_id","event":"validation_failed","desc":"Dependency cycle detected: {task_id} → TASK-A → TASK-B → {task_id}","cycle_path":"[list of task IDs in cycle]"}
     ```
   - Present cycle path to user for resolution
   - User must break cycle by:
     - Removing a different dependency
     - Re-sequencing tasks
     - Splitting tasks to break circular relationship
   - Why MUST: Cycles would deadlock the system

4. **If No Cycle** ⚠️ **MUST**
   - Proceed with WF-LINK to add dependency
   - Log success (optional):
     ```json
     {"ts":"ISO8601","user":"user_id","event":"cycle_check_passed","desc":"No circular dependencies detected"}
     ```
   - Why MUST: Safe to proceed

**Manual DFS Algorithm** (Human-Executable):

```
Given: task_id, new_dependency

Step 1: Check immediate self-loop
  If new_dependency == task_id → CYCLE (stop)

Step 2: Trace dependency chain
  current = new_dependency
  path = [task_id, new_dependency]
  
  While current has dependencies:
    For each dep in current's depends_on:
      If dep == task_id → CYCLE (stop, report path)
      If dep in path → CYCLE (stop, report path)
      Add dep to path
      Set current = dep
      Repeat
  
Step 3: If completed without finding task_id in chain → NO CYCLE
```

**Example Scenarios**:

**Scenario 1: Cycle Detected**
```
TASK-0042 depends_on: [TASK-0037]
TASK-0037 depends_on: [TASK-0055]
TASK-0055 depends_on: [TASK-0089]

Attempting to add: TASK-0089 depends_on: [TASK-0042]

Trace:
  TASK-0042 → TASK-0037 → TASK-0055 → TASK-0089 → TASK-0042 ❌ CYCLE!

Path: TASK-0042 → TASK-0037 → TASK-0055 → TASK-0089 → TASK-0042
```

**Scenario 2: No Cycle**
```
TASK-0042 depends_on: [TASK-0037]
TASK-0037 depends_on: [TASK-0030]
TASK-0055 depends_on: [TASK-0037]

Attempting to add: TASK-0042 depends_on: [TASK-0055]

Trace from TASK-0055:
  TASK-0055 → TASK-0037 → TASK-0030 (end)
  
TASK-0042 not in chain → NO CYCLE ✓
```

**Skip Guidance**:
- **MUST steps**: Cycle detection is required for graph integrity
- No flexible steps - this is a safety check

**Required Ledger Events**:
- Cycle detected: `{"ts":"ISO8601","user":"user_id","event":"validation_failed","desc":"Dependency cycle detected","cycle_path":"[...]"}`
- No cycle (optional): `{"ts":"ISO8601","user":"user_id","event":"cycle_check_passed","desc":"No circular dependencies"}`

**Note**: This is a manual process for the "perfect human" spec. When implementing CLI/API, this becomes an automated graph algorithm.
