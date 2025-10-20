---
type: workflow
stage: archived
parent: ../tasks.workflow.md
revisit_after_executions: 25
last_revisited: 2025-10-18
criticality: flexible
---

# Archived Workflows

This file contains workflows specific to the archived stage.

## WF9 - Final Closure

**Criticality**: flexible (can adapt as needed, all skips are valuable signals)

**Purpose**: Final verification and task closure. Tasks in archived serve as permanent record.

**Steps**:

1. **Final Verification** ✓ Flexible
   - Verify all acceptance criteria were met
   - Confirm all follow-up tasks created and logged in ledger
   - Verify learning completed or skip documented
   - Confirm knowledge captured (technical and/or process)
   - Ensure no pending work or rework required
   - Verify ledger contains complete event history
   - Skip reason example: "DRI verified all complete during learning stage"

2. **Close Task** ⚠️ **MUST**
   - Log event: `{"ts":"ISO8601","user":"user_id","event":"closed","desc":"Task closed and archived"}`
   - Task becomes **read-only**
   - Task remains in `archived/` as permanent record
   - Why MUST: Marks definitive task closure in ledger

**Final Verification Checklist**:
- [ ] All acceptance criteria met
- [ ] All follow-up tasks created and logged in ledger
- [ ] Learning stage completed
- [ ] Knowledge captured (technical and/or process)
- [ ] No pending work or rework required
- [ ] Ledger contains complete event history
- [ ] Task folder in archived/
- [ ] All indexes updated correctly

**Skip Guidance**:
- **MUST steps**: Do not skip - protects system integrity
- **Flexible steps**: Skip with judgment
  - Document skip reasons when helpful
  - Trust DRI to verify appropriately
  - **Every skip is feedback**

**Required Ledger Events**:
- Task closed: `{"ts":"ISO8601","user":"user_id","event":"closed","desc":"Task closed and archived"}`

**Note**: Tasks in archived stage are **read-only** and serve as permanent record with complete provenance trail. They are available for learning pattern collection (WF8) and post-mortem analysis.
