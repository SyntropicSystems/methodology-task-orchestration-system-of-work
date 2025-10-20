---
type: workflow_overview
parent: README.md
revisit_after_executions: 25
last_revisited: 2025-10-18
---

# Task Management System Workflows

This file provides an overview of all workflows in the task management system. Each workflow is implemented in its respective stage folder.

## Workflow Overview

The task management system consists of nine primary workflows across seven stages:

### Inbox Stage
- **WF1 - Create Task Package**: Create new task with all required files
- **WF2 - Verify Task Package and Move to Backlog**: Verify completeness and move to backlog

**Details**: See [inbox/inbox.workflow.md](inbox/inbox.workflow.md)

### Backlog Stage
- **WF3 - Prioritization and Moving Tasks to Next**: Prioritize and schedule tasks

**Details**: See [backlog/backlog.workflow.md](backlog/backlog.workflow.md)

### Next Stage
- **WF4 - Move to Current**: Start active implementation

**Details**: See [next/next.workflow.md](next/next.workflow.md)

### Current Stage
- **WF5 - Move to Done**: Complete implementation and verification

**Details**: See [current/current.workflow.md](current/current.workflow.md)

### Integration Stage
- **WF6 - Integration and Technical Knowledge Capture**: Merge, release, and capture technical learnings

**Details**: See [integration/integration.workflow.md](integration/integration.workflow.md)

### Learning Stage
- **WF7 - System-of-Work Review and Learning**: Double-loop learning on process effectiveness
- **WF8 - Learning Pattern Collection**: Periodic analysis of learning patterns (run periodically)

**Details**: See [learning/learning.workflow.md](learning/learning.workflow.md)

### Archived Stage
- **WF9 - Final Closure**: Final verification and task closure

**Details**: See [archived/archived.workflow.md](archived/archived.workflow.md)

## Cross-Cutting Workflows (Global)

These workflows support the nine primary stage workflows and are called from multiple stages:

- **WF-VALIDATE-MOVE** - Pre-flight validation before stage transitions → See `workflows/validation.workflow.md`
- **WF-CASCADE-CANCEL, WF-CASCADE-PAUSE** - Handle dependent tasks when state changes → See `workflows/state-management.workflow.md`
- **WF-SPLIT, WF-MERGE, WF-LINK, WF-DETECT-CYCLES** - Graph operations and dependency management → See `workflows/graph-operations.workflow.md`
- **WF-HANDOFF** - Transfer task ownership with acceptance/rejection → See `workflows/handoff.workflow.md`
- **P0 Fast-Track Rules** - Expedited workflow for critical priorities → See `workflows/priority-effort.workflow.md`

**Integration Points**:
- WF-VALIDATE-MOVE is **embedded** in WF3 (backlog → next) and WF4 (next → current)
- WF-CASCADE-CANCEL is **triggered** automatically by WF-CANCEL when canceling tasks with dependents
- WF-DETECT-CYCLES is **required** before adding dependencies via WF-LINK
- P0 Fast-Track modifies the standard stage progression (skips `next` stage)

---

## Operating System Layer

The Operating System Layer provides the infrastructure for safe speed, observability, agent integration, and continuous improvement. These specifications transform the workflow spine from "ready for automation" to "ready for scale."

### Flow Control & Observability

- **WIP Policy** (`wip.policy.md`) - Work-in-progress limits, SLO targets, queue discipline, andon rules
  - Defines hard/soft WIP limits per stage
  - Service Level Objectives for each stage transition
  - P0 fast-track displacement rules
  - Flow control signals (yellow/red flags)

- **Metrics Schema** (`metrics.schema.md`) - Event-sourced metrics for data-driven learning
  - Core duration metrics (lead time, flow time, blocked time)
  - Flow metrics (throughput, WIP, arrival rate, rework rate)
  - Derived ratios (flow efficiency, predictability, SLO compliance)
  - Computation formulas from ledger events

- **Reports Specification** (`reports.spec.md`) - Standard reports for different audiences
  - Weekly Flow Report (system health, capacity planning)
  - Blockers Report (dependency coordination)
  - Rework & Rollback Report (quality signals)
  - P0 Postmortem Summary (critical incident learning)
  - Learning View Report (process evolution patterns)

### Agent Architecture

- **Query Specification** (`query.spec.md`) - Consistent data access interface
  - Canonical queries (get_task, get_blocked_tasks, get_wip_by_stage, etc.)
  - Multiple interface options (CLI, file-based, API, SQL)
  - Query composition and performance considerations
  - Makes system queryable by humans and agents consistently

- **Worker Contract** (`worker.contract.md`) - Agent interface specification ⭐
  - Core capabilities (plan, validate, execute, review, summarize, propose_move)
  - Idempotency rules (propose events, don't apply directly)
  - Worker types (planning, validation, execution, review, integration, learning)
  - Capability manifests and error contracts
  - **Makes "agent-agnostic" architecture real, not aspirational**

### Extensibility & Scale

- **Events Schema** (`events.schema.md`) - Event versioning and extension hooks
  - Event envelope format with version field
  - Versioning rules (minor vs major changes)
  - Hook points (on_validate_pass, on_move, on_blocked, on_slo_missed, etc.)
  - Hook contract for custom logic injection
  - Backward-compatible event evolution

- **Namespaces** (`namespaces.md`) - Multi-team scaling support
  - Namespace ID scheme ({NAMESPACE}-{ID})
  - Folder structure options (unified, per-namespace, hybrid)
  - Cross-namespace dependencies and cycle detection
  - Per-namespace WIP limits and permissions
  - Migration strategies for organizational growth

- **Console Views** (`console.view.md`) - Operator situational awareness
  - Now View (command center for immediate attention)
  - Flow View (system health and capacity)
  - Blockers View (dependency coordination)
  - Rework View (quality signals)
  - Learning View (process evolution)

- **Flywheel** (`flywheel.md`) - Automation promotion framework
  - 5 automation maturity levels (Human → Human-AI → AI-Human → AI+Spot → Automated)
  - 7-stage flywheel loop (Capture → Template → Validate → Delegate → Instrument → Learn → Automate)
  - Evidence-based promotion criteria
  - Human role evolution (hands-on → monitoring → strategic)
  - Demotion and rollback procedures

**Integration with Workflows**:
- WIP limits enforced in WF4 (next → current)
- Metrics computed from all workflow event logs
- Queries power reports and console views
- Workers execute workflows with human approval gates
- Hooks extend workflows without core modifications
- Namespaces scale workflows across teams
- Flywheel automates workflow execution over time

---

## Workflow Execution Notes

- All workflows require human sign-off before moving tasks between stages
- Each workflow step must be logged in the task ledger with timestamp and user ID
- Workflows follow single source of truth principle - no duplication between stages
- Index files must be kept synchronized with task movements
- Cross-cutting workflows are invoked as needed to maintain system integrity
