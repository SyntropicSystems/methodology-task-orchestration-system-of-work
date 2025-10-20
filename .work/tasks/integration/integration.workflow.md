---
type: workflow
stage: integration
parent: ../tasks.workflow.md
revisit_after_executions: 25
last_revisited: 2025-10-18
criticality: flexible
---

# Integration Workflows

This file contains workflows specific to the integration stage.

## WF6 - Integration and Technical Knowledge Capture

**Criticality**: flexible (can adapt as needed, all skips are valuable signals)

**Purpose**: Merge, release, integrate completed work, capture technical knowledge, and create follow-up tasks.

**Steps**:

1. **Release/Deploy** ⚠️ **MUST**
   - Execute merge, release, or deployment as appropriate for task type
   - Verify successful integration
   - Log event: `{"ts":"ISO8601","user":"user_id","event":"integrated","desc":"Task integrated and released"}`
   - Why MUST: Ensures task deliverables are properly released

2. **Technical Knowledge Capture** ✓ Flexible
   - Document key learnings from **implementation** (not process)
   - Identify architectural insights or implications
   - Note technical patterns that could inform future tasks
   - Update relevant knowledge base or documentation
   - Capture technical context that would benefit others
   - Skip reason example: "Standard implementation, no novel technical patterns"

3. **Create Follow-up Tasks** ✓ Flexible (as needed)
   - Identify any additional **technical** work needed
   - Examples: Additional docs, refactoring, new features identified
   - Create new task packages in `inbox/` for each follow-up
   - Log in current task ledger: `{"ts":"ISO8601","user":"user_id","event":"followup_created","desc":"Follow-up task TASK-XXX created for [reason]"}`
   - This maintains complete traceable graph with provenance
   - Skip reason example: "No follow-up work identified"

4. **Move to Learning** ⚠️ **MUST**
   - Move task folder from `integration/` to `learning/`
   - Update indexes:
     - Cut entry from `integration/INDEX.md`
     - Paste entry into `learning/INDEX.md`
     - Update root `INDEX.md` with new status
   - Log event: `{"ts":"ISO8601","user":"user_id","event":"moved_to_learning","desc":"Task moved to learning stage"}`
   - Why MUST: Prevents task orphaning, maintains index consistency

**Integration Checklist**:
- [ ] Task integrated/released successfully
- [ ] Technical knowledge captured (or skip documented)
- [ ] All follow-up tasks created (or none needed)
- [ ] Follow-ups logged in ledger with provenance
- [ ] Task folder moved to `learning/`
- [ ] All three INDEX.md files updated
- [ ] Ledger events logged

**Skip Guidance**:
- **MUST steps**: Do not skip - protects system integrity
- **Flexible steps**: Skip with judgment
  - Documenting skip reason helps everyone
  - Not documenting: Still trusted
  - **Every skip is feedback** - signals workflow effectiveness

**Required Ledger Events**:
- Integration complete: `{"ts":"ISO8601","user":"user_id","event":"integrated","desc":"Task integrated and released"}`
- Knowledge captured: `{"ts":"ISO8601","user":"user_id","event":"knowledge_captured","desc":"Technical knowledge documented"}` (or skip reason)
- Follow-up created: `{"ts":"ISO8601","user":"user_id","event":"followup_created","desc":"Follow-up task TASK-XXX created"}`
- Move to learning: `{"ts":"ISO8601","user":"user_id","event":"moved_to_learning","desc":"Task moved to learning stage"}`
