---
type: stage_documentation
stage: next
parent: ../README.md
last_revisited: 2025-10-18
---

# Next

## Purpose

The next queue contains prioritized tasks that are ready to be worked on. Tasks are organized by effort and ordered by priority within each effort.

## Entry Criteria

- Task prioritized with clear priority number
- Dependencies identified and ordered
- Task grouped within appropriate effort
- Priority numbers are consecutive and monotonic within effort

## Exit Criteria

- Task picked up for active work
- DRI assigned and committed to implementation
- Task ready to move to current
- Implementation event logged in ledger

## Governance Files

| File | Purpose |
|------|---------|
| README.md | This file - stage context and criteria |
| INDEX.md | Tracking tasks currently in next queue |
| next.workflow.md | Workflow definitions for this stage (WF4) |

## References

- Parent: [Task Management System](../README.md)
- Workflows: [next.workflow.md](next.workflow.md)
