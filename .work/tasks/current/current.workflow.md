---
type: workflow
stage: current
parent: ../tasks.workflow.md
revisit_after_executions: 25
last_revisited: 2025-10-18
criticality: flexible
---

# Current Workflows

This file contains workflows specific to the current stage.

## WF5 - Move to Done

**Criticality**: flexible (can adapt verification approach, all adaptations are signals)

**Purpose**: Complete implementation, verify, and move task to integration.

**Steps**:

1. **Verify Implementation** ✓ Flexible
   - Review all acceptance criteria from task card
   - Test implementation against each criterion
   - Verify all expected outputs are delivered
   - Check documentation is created/updated as required

2. **If Verification Passes**:
   - Log event: `{"ts":"ISO8601","user":"user_id","event":"implemented","desc":"Implementation completed"}` ⚠️ **MUST**
   - Move task folder from `current/` to `integration/` ⚠️ **MUST**
   - Update indexes: ⚠️ **MUST**
     - Cut entry from `current/INDEX.md`
     - Paste entry into `integration/INDEX.md`
     - Update root `INDEX.md` with new status
   - Log event: `{"ts":"ISO8601","user":"user_id","event":"verified","desc":"Task verified and moved to integration"}`

3. **If Verification Fails**:
   - Document specific issues found ✓ Flexible
   - Update or extend acceptance criteria if needed ✓ Flexible
   - Log event: `{"ts":"ISO8601","user":"user_id","event":"rework_required","desc":"Verification failed: [reason]"}` ⚠️ **MUST**
   - Move task folder back to `next/` ⚠️ **MUST**
   - Update all indexes accordingly ⚠️ **MUST**

**Skip Guidance**:
- **MUST steps**: Logging state transitions, moving folders, updating indexes - protects system integrity
- **Flexible steps**: Verification depth, criteria updates - adapt as needed
- **Every skip/adaptation signals** workflow effectiveness

**Verification Checklist**:
- [ ] All acceptance criteria met
- [ ] All expected outputs delivered
- [ ] Documentation updated
- [ ] Independent verification completed
- [ ] Task folder moved to appropriate stage
- [ ] All three INDEX.md files updated
- [ ] Ledger events logged

**Required Ledger Events**:
- Implementation complete: `{"ts":"ISO8601","user":"user_id","event":"implemented","desc":"Implementation completed"}`
- Verification pass: `{"ts":"ISO8601","user":"user_id","event":"verified","desc":"Task verified and moved to integration"}`
- Verification fail: `{"ts":"ISO8601","user":"user_id","event":"rework_required","desc":"Verification failed, moved back to next"}`
