---
type: stage_documentation
stage: inbox
parent: ../README.md
last_revisited: 2025-10-18
---

# Inbox

## Purpose

The inbox is the entry point for all new tasks. Tasks in the inbox are awaiting verification before moving to the backlog.

## Entry Criteria

- Task package created with all required files
- Task follows the naming schema `TASK-{id}-{name}`
- Initial event logged in task ledger

## Exit Criteria

- Task package verified as complete and correct
- All required files present and properly formatted
- Context is hermetic and sufficient for implementation
- Acceptance criteria are testable and clear
- Task ready to move to backlog for prioritization

## Governance Files

| File | Purpose |
|------|---------|
| README.md | This file - stage context and criteria |
| INDEX.md | Tracking tasks currently in inbox |
| inbox.workflow.md | Workflow definitions for this stage (WF1, WF2) |

## References

- Parent: [Task Management System](../README.md)
- Workflows: [inbox.workflow.md](inbox.workflow.md)
