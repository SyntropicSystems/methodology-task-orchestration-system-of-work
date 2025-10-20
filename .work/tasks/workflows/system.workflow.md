---
type: workflow
stage: system
parent: ../tasks.workflow.md
revisit_after_executions: 25
last_revisited: 2025-10-18
criticality: must
---

# System Workflows

This file contains system-level workflows that maintain task system integrity.

## WF0 - Index Sanity Check

**Criticality**: must (protects system integrity)

**Purpose**: Validate system consistency before any task movement to prevent index drift, orphaned tasks, and broken dependencies.

**When**: ⚠️ **MUST** run before executing WF2, WF3, WF4, WF5, WF6, WF7 (any workflow that moves tasks between stages)

**Steps**:

1. **Scan Task Folders** ⚠️ **MUST**
   - List all `TASK-*` directories across all stage folders
   - Extract task IDs from folder names
   - Why MUST: Establishes ground truth of what tasks exist

2. **Read All Indexes** ⚠️ **MUST**
   - Read all stage `INDEX.md` files (inbox, backlog, next, current, integration, learning, archived)
   - Read root `INDEX.md`
   - Parse task entries from each
   - Why MUST: Establishes what indexes claim exists

3. **Verify Rule 1: ID Uniqueness** ⚠️ **MUST**
   - Check that no two task folders have the same task_id
   - If duplicates found: HALT with error listing duplicate IDs
   - Why MUST: Prevents ID collisions that break provenance

4. **Verify Rule 2: Index-Folder Parity** ⚠️ **MUST**
   - For each task folder: verify exactly one entry exists in its stage INDEX.md
   - For each task folder: verify exactly one entry exists in root INDEX.md
   - For each index entry: verify corresponding task folder exists
   - If mismatch found: HALT with error listing orphaned tasks or phantom entries
   - Why MUST: Ensures indexes are accurate map to reality

5. **Verify Rule 3: Stage Congruency** ⚠️ **MUST**
   - For each task: read `stage` field from task card frontmatter
   - Verify it matches the folder location (e.g., task in `current/` has `stage: current`)
   - If mismatch found: HALT with error listing incongruent tasks
   - Why MUST: Prevents tasks from claiming wrong stage

6. **Verify Rule 4: Status Congruency** ⚠️ **MUST**
   - For each task: compare `status` and `stage` in frontmatter vs INDEX.md columns
   - Verify they match exactly
   - If mismatch found: HALT with error listing discrepancies
   - Why MUST: Ensures single source of truth

7. **Verify Rule 5: Dependency Existence** ⚠️ **MUST**
   - For each task: read `depends_on` array from frontmatter
   - For each dependency ID: verify a task folder with that ID exists somewhere in the system
   - If missing dependency found: HALT with error listing broken dependencies
   - Why MUST: Prevents dangling dependency references

8. **Log Result** ⚠️ **MUST**
   - If all checks pass: Log to each moved task's ledger: `{"ts":"ISO8601","user":"user_id","event":"index_synced","desc":"System validation passed"}`
   - If any check fails: Log: `{"ts":"ISO8601","user":"user_id","event":"index_drift_detected","desc":"[specific error]"}`
   - Why MUST: Maintains audit trail of validation

**Validation Checklist**:
- [ ] All task IDs are unique
- [ ] Every task folder has exactly one stage index entry
- [ ] Every task folder has exactly one root index entry
- [ ] No phantom index entries (entries without folders)
- [ ] All task frontmatter `stage` fields match folder locations
- [ ] All index Status/Flags match task frontmatter
- [ ] All dependencies in `depends_on` arrays exist

**On Failure**: DO NOT proceed with task movement. Fix the inconsistency first, then retry.

**Required Ledger Events**:
- Success: `{"ts":"ISO8601","user":"user_id","event":"index_synced","desc":"System validation passed"}`
- Failure: `{"ts":"ISO8601","user":"user_id","event":"index_drift_detected","desc":"[error description]"}`

---

## WF-UPDATE-CONTENT - Content Versioning

**Criticality**: flexible (can adapt versioning approach)

**Purpose**: Update task goal, context, or acceptance criteria after task has left inbox stage, while maintaining complete history.

**When**: Need to change task requirements, context, or acceptance criteria for a task that's already been verified

**Immutability Principle**: Once a task leaves inbox, its core files should be treated as immutable versions to preserve provenance.

**Steps**:

1. **Determine What Changed** ✓ Flexible
   - Identify which file needs updating: task card, context, or both
   - Document the reason for the change
   - Skip reason example: "Minor typo fix, not worth versioning"

2. **Create Version Backup** ⚠️ **MUST**
   - Do NOT edit the original file in place
   - Rename original file to include version suffix:
     - `{id}-{name}.task.md` → `{id}-{name}.task.v1.md`
     - `{id}.context.md` → `{id}.context.v1.md`
   - Why MUST: Preserves complete history

3. **Create New Version** ⚠️ **MUST**
   - Create new file with original name containing updated content
   - This becomes v2 (or v3, v4, etc. if multiple updates)
   - Maintain all frontmatter, only update content sections
   - Why MUST: Maintains expected file paths while preserving history

4. **Update Frontmatter** ✓ Flexible
   - Update `last_updated` date if you add one
   - Optionally add version note in frontmatter
   - Skip reason example: "Using filename versioning only"

5. **Log Event** ⚠️ **MUST**
   - Add to task ledger: `{"ts":"ISO8601","user":"user_id","event":"content_updated","file":"{filename}","new_version":"v2","reason":"[description]"}`
   - Why MUST: Maintains provenance of all changes

**Versioning Example**:

Before update:
```
TASK-0042-feature/
├── 0042-feature.task.md
├── 0042.context.md
└── 0042.ledger.jsonl
```

After update:
```
TASK-0042-feature/
├── 0042-feature.task.v1.md     # Original
├── 0042-feature.task.md         # Updated (v2)
├── 0042.context.md              # Unchanged
└── 0042.ledger.jsonl            # New event logged
```

**Skip Guidance**:
- **MUST steps**: File versioning and logging maintain provenance
- **Flexible steps**: Can adapt versioning details
- Minor typos or formatting: Use judgment on whether to version

**Required Ledger Event**:
```json
{"ts":"2025-10-18T10:30:00Z","user":"@alice","event":"content_updated","file":"0042-feature.task.md","new_version":"v2","reason":"Stakeholder changed requirements: added third acceptance criterion"}
```

**Note**: This manual versioning is intensive but provides complete audit trail in a file-based system. Future automation can implement this more elegantly.
