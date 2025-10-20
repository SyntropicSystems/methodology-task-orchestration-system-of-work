---
type: workflow
parent: ../tasks.workflow.md
revisit_after_executions: 25
last_revisited: 2025-10-18
criticality: must
---

# Handoff Workflow

## WF-HANDOFF - Transfer Task Ownership

**Criticality**: must

**Purpose**: Transfer task ownership (DRI) from one person to another with explicit acceptance or rejection.

**When**:
- Team restructuring
- DRI leaving team
- Load balancing
- Skill match (different phase needs different expertise)

**Important**: This is a **two-step process** with explicit acceptance required to maintain DRI accountability.

**Steps**:

### Phase 1: Initiation

1. **Identify New DRI** ⚠️ **MUST**
   - Confirm new person is willing to consider responsibility
   - DO NOT proceed without verbal/written pre-agreement
   - Why MUST: DRI accountability principle requires willing acceptance

2. **Document Knowledge Transfer Plan** ⚠️ **MUST**
   - Schedule synchronous hand-off session (highly recommended)
   - Prepare hand-off checklist covering:
     - Current state of work
     - What's completed
     - What remains
     - Known blockers or issues
     - Key decisions made
   - Why MUST: Smooth transition requires thorough knowledge transfer

3. **Log Hand-off Initiation** ⚠️ **MUST**
   - Add to task ledger:
     ```json
     {"ts":"ISO8601","user":"{old_dri}","event":"handoff_initiated","from":"@{old_dri}","to":"@{new_dri}","reason":"[reason for handoff]"}
     ```
   - Why MUST: Tracks start of hand-off process

### Phase 2: Knowledge Transfer

4. **Update Context File** ⚠️ **MUST**
   - Add hand-off section to `{id}.context.md` using template below
   - Document current state, completed work, remaining work, gotchas
   - Why MUST: Ensures new DRI has complete written context

5. **Conduct Knowledge Transfer** ⚠️ **MUST** (highly recommended)
   - Schedule and conduct synchronous session (30-60 min typical)
   - Walk through context document
   - Answer questions
   - Ensure new DRI understands scope, state, and acceptance criteria
   - Why MUST: Reduces risk of lost context or misunderstanding

### Phase 3: Accept or Reject

6. **New DRI Decision** ⚠️ **MUST**
   
   **Option A: Accept Responsibility**
   - New DRI explicitly accepts ownership
   - Reviews task and confirms understanding
   - Log acceptance:
     ```json
     {"ts":"ISO8601","user":"{new_dri}","event":"handoff_accepted","from":"@{old_dri}","to":"@{new_dri}","reason":"Accepted ownership after knowledge transfer"}
     ```
   - Update task frontmatter: `current_dri: "@{new_dri}"`
   - Add to frontmatter history (optional): `previous_dris: ["@{old_dri}"]`
   - Update DRI column in stage `INDEX.md`
   - Update DRI column in root `INDEX.md`
   
   **Option B: Reject Handoff**
   - New DRI declines responsibility with clear reason
   - Log rejection:
     ```json
     {"ts":"ISO8601","user":"{new_dri}","event":"handoff_rejected","from":"@{old_dri}","to":"@{new_dri}","reason":"[specific reason - e.g., insufficient capacity, skills mismatch, unclear requirements]"}
     ```
   - Task remains with old DRI: `current_dri` unchanged
   - Old DRI must:
     - Find alternate DRI, OR
     - Keep task, OR
     - Pause/cancel task if no suitable DRI available
   - Consider if task should be paused (WF-PAUSE) while finding new DRI
   
   Why MUST: DRI accountability requires explicit acceptance, rejection must be documented

**Hand-off Context Template** (add to `{id}.context.md`):

```markdown
## Hand-off: {Old DRI} → {New DRI} on {Date}

### Current State
- Status: [active|blocked|paused]
- Stage: [current stage]
- Progress: [X% complete, or specific milestones achieved]

### Work Completed
- [What has been done]
- [Key decisions made]
- [Files modified/created]

### Work Remaining
- [What needs to be done]
- [Acceptance criteria not yet met]
- [Estimated effort remaining]

### Blockers & Dependencies
- [Current blockers if any]
- [Dependencies on other tasks]
- [External dependencies]

### Context & Gotchas
- [Important background information]
- [Non-obvious dependencies or constraints]
- [Things to watch out for]
- [Technical debt or shortcuts taken]

### Questions for New DRI
- [Any open questions]
- [Clarifications needed]
- [Assumptions to validate]
```

**Skip Guidance**:
- **MUST steps**: Documentation, two-step acceptance, and ledger events maintain accountability
- Synchronous knowledge transfer session is **highly recommended** but can be adapted if both parties agree to async transfer with thorough documentation

**Required Ledger Events**:
- Initiation: `{"ts":"ISO8601","user":"@old_user","event":"handoff_initiated","from":"@old_user","to":"@new_user","reason":"[reason]"}`
- Acceptance: `{"ts":"ISO8601","user":"@new_user","event":"handoff_accepted","from":"@old_user","to":"@new_user","reason":"Accepted ownership after knowledge transfer"}`
- OR Rejection: `{"ts":"ISO8601","user":"@new_user","event":"handoff_rejected","from":"@old_user","to":"@new_user","reason":"[specific reason]"}`

**Handoff Checklist**:
- [ ] New DRI identified and willing
- [ ] Hand-off section added to context file
- [ ] Knowledge transfer session conducted (or async documentation complete)
- [ ] New DRI has reviewed task card, context, and acceptance criteria
- [ ] New DRI explicitly accepted or rejected
- [ ] Appropriate ledger event logged
- [ ] If accepted: DRI updated in frontmatter and indexes
- [ ] If rejected: Next steps determined (find alternate, pause, etc.)
