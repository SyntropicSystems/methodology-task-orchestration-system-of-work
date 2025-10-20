---
type: workflow
stage: learning
parent: ../tasks.workflow.md
revisit_after_executions: 25
last_revisited: 2025-10-18
criticality: flexible
---

# Learning Workflows

This file contains workflows specific to the learning stage.

## WF7 - System-of-Work Review and Learning

**Criticality**: flexible (can adapt as needed, all skips are valuable signals)

**Purpose**: Conduct double-loop learning - reflect on **how we worked**, not what we built.

**Steps**:

1. **System-of-Work Review** ✓ Flexible
   - Review task execution from creation through integration
   - Assess: Was the process effective?
   - Identify: What friction occurred in workflows?
   - Note: What worked well / what didn't?
   - Skip reason example: "Simple task, no process insights to capture"

2. **Create Learning Document** ✓ Flexible
   - File: `{id}.learning.md` (recommended, not required)
   - Use template provided below
   - Log: `{"ts":"ISO8601","user":"user_id","event":"learning_documented","desc":"Learning document created"}`
   - Skip reason example: "Task followed standard pattern, no unique learnings"
   - Note: Skipping is okay, but documenting helps everyone improve

3. **Move to Archived** ⚠️ **MUST**
   - Move task folder from `learning/` to `archived/`
   - Update indexes:
     - Cut entry from `learning/INDEX.md`
     - Paste entry into `archived/INDEX.md`
     - Update root `INDEX.md` with new status
   - Log: `{"ts":"ISO8601","user":"user_id","event":"moved_to_archived","desc":"Task moved to archived"}`
   - Why MUST: Prevents task orphaning, maintains index consistency

**Learning Document Template**:

```markdown
# System of Work Learning

## Process Execution
[What worked / didn't work in following workflows?]
[Was information available when needed?]
[Were stage transitions smooth?]

## Workflow Effectiveness
[Which workflows were helpful?]
[Which created friction or felt unnecessary?]
[Were any steps unclear or confusing?]

## Improvements Identified
- [ ] Potential process improvement 1
- [ ] Potential process improvement 2

## Review Triggers
[Should any workflows be reviewed based on this execution?]
[After N similar experiences, what pattern emerges?]

## Meta-Patterns
[Patterns about how we work, not what we built]
```

**Skip Guidance**:
- **MUST steps**: Do not skip - protects system integrity
- **Flexible steps**: Skip with judgment
  - Documenting skip reason helps: Post-mortems, collective learning, workflow evolution
  - Not documenting: Still trusted, but missed improvement opportunity
  - **Every skip is feedback** - signals potential for workflow improvement

**Verification Checklist**:
- [ ] System-of-work review completed (or documented skip reason)
- [ ] Learning document created (or skip decision logged)
- [ ] Task folder moved to archived/
- [ ] All three INDEX.md files updated
- [ ] Ledger events logged

**Required Ledger Events**:
- Learning documented: `{"ts":"ISO8601","user":"user_id","event":"learning_documented","desc":"Learning document created"}`
- OR Learning skipped: `{"ts":"ISO8601","user":"user_id","event":"learning_skipped","desc":"[reason]"}` (optional but helpful)
- Move to archived: `{"ts":"ISO8601","user":"user_id","event":"moved_to_archived","desc":"Task moved to archived"}`

---

## WF8 - Learning Pattern Collection

**Criticality**: flexible

**Purpose**: Periodic collection and analysis of learning patterns across archived tasks.

**When**: Run periodically (not per-task) when enough learnings have accumulated.

**Steps**:

1. **Collect Learning Patterns** ✓ Flexible
   - Review learning documents across multiple archived tasks
   - Identify recurring themes and patterns
   - Note systemic issues mentioned multiple times
   - Skip reason example: "Not enough tasks archived yet for meaningful patterns"

2. **Analyze Effectiveness Signals** ✓ Flexible
   - Look for workflow steps frequently skipped
   - Identify consistent friction points
   - Note which workflows receive positive feedback
   - Count references to specific workflow issues

3. **Create Analysis Tasks** ✓ Flexible (as needed)
   - If significant patterns emerge, create analysis task in inbox
   - Example: "Analyze WF4 friction - mentioned in 5 learning docs"
   - Link to source tasks for provenance
   - Log: `{"ts":"ISO8601","user":"user_id","event":"learning_analysis_created","desc":"Created TASK-XXX to analyze [pattern]"}`

**Collection Triggers**:
- After every N archived tasks (flexible, start with 10)
- When specific workflow mentioned repeatedly
- When team member requests pattern review
- Quarterly review (flexible timing)

**Output**: Analysis tasks in inbox for deeper investigation and potential workflow improvements.

**Philosophy**: This is where **signal becomes action** - converting real-world feedback into systematic improvements.
