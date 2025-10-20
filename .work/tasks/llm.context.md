---
title: "Task Management System - LLM Context"
description: "System-level documentation for event-driven task management"
version: "1.0"
last_updated: "2025-10-18"
target_audience: "AI agents and developers"
purpose: "Single source of truth for task management system architecture and philosophy"
---

# Task Management System - LLM Context

**Purpose**: This file documents the task management system architecture, philosophy, and mental models. For implementation details (workflows, schemas, templates), see the referenced files below.

**If you're new to this system, start here.** It will route you to everything else.

---

## 1. System Philosophy

### Purpose
Event-driven task tracking in a dependency graph with full event log, provenance, and complete trace graph.

### Core Principles

1. **Emergence Over Planning**: We evolve workflows through signal intelligence - doing what serves us, removing what doesn't
2. **Progressive Automation**: Human → Human-AI → AI-Human → AI → Automation (move deterministic work down the chain)
3. **Effectiveness as Truth**: Workflows have review triggers after N executions (not time-based)
4. **Human Sign-off**: Tasks moving to next, integration, or archived require human approval
5. **DRI Accountability**: One person responsible end-to-end for task orchestration and results
6. **Single Source of Truth**: No duplication - everything references its canonical location

### Automation Philosophy

- ✅ Automate deterministic processes early
- ✅ Use AI for bounded determinism
- ✅ Keep humans for one-offs and high-value judgment
- ✅ Invest smartly - don't over-automate prematurely

---

## 2. Architecture & Mental Model

### Graph-Based Structure

Tasks form a **directed acyclic graph (DAG)**:
- **Nodes**: Tasks (with full provenance)
- **Edges**: Dependencies between tasks
- **Properties**: Each node has complete event history via append-only ledger

### Stage-Based Lifecycle

Tasks flow through seven stages:

```
inbox → backlog → next → current → integration → learning → archived
```

Each stage has:
- Entry criteria (what brings tasks here)
- Exit criteria (what moves tasks forward)
- Dedicated workflows for transitions

### Event Sourcing

Every task state change is recorded in an **append-only ledger** (`{id}.ledger.jsonl`):
- Immutable history
- Complete provenance
- Full traceability
- Enables replay and audit

### Knowledge Graph

Documents use **YAML frontmatter** to create a browseable graph:
- Type classification (workflow, index, stage_documentation, etc.)
- Parent/child relationships
- Revisit triggers for workflows
- Last updated timestamps

---

## 3. Core Concepts

### Task Package

A task is a **folder** containing exactly three files:
- `{id}-{name}.task.md` - Task card (goal, inputs, outputs, acceptance criteria)
- `{id}.context.md` - Hermetic context (everything needed to implement)
- `{id}.ledger.jsonl` - Append-only event log

**Key property**: Task package is **self-contained** and **portable**.

### Workflow

A workflow is a **defined procedure** for moving tasks between stages or performing operations. Each workflow:
- Has a unique ID (WF1, WF2, etc.)
- Belongs to a specific stage
- Has clear steps and checklists
- Specifies required ledger events
- Has a revisit trigger (review after N executions)

### Stage

A stage is a **phase in the task lifecycle**. Each stage:
- Has a dedicated folder with README.md, INDEX.md, {stage}.workflow.md
- Defines entry and exit criteria
- Contains tasks currently in that phase
- Has stage-specific workflows

### Index

An index is a **Markdown table** tracking tasks at a specific level:
- Root INDEX.md: All tasks across all stages
- Stage INDEX.md: Tasks in that specific stage
- Always kept synchronized with task movements

### Ledger

The ledger is an **append-only JSONL file** recording all task events:
- One JSON object per line
- Fields: timestamp, user, event type, description
- Immutable - never edited, only appended
- Provides complete task history

---

## 4. File Structure Pattern

```
.work/tasks/
├── README.md                    # System overview & schemas
├── INDEX.md                     # Root task index
├── tasks.workflow.md            # Workflow overview
├── llm.context.md              # This file
│
├── {stage}/                     # One folder per stage
│   ├── README.md               # Stage purpose, criteria, references
│   ├── INDEX.md                # Tasks in this stage
│   ├── {stage}.workflow.md     # Stage-specific workflows
│   └── TASK-{id}-{name}/       # Task packages
│       ├── {id}-{name}.task.md
│       ├── {id}.context.md
│       └── {id}.ledger.jsonl
```

### File Types & Purpose

| File | Purpose | References |
|------|---------|------------|
| README.md | System overview & minimal schemas | This is the entry point |
| INDEX.md | Task tracking tables | Updated by workflows |
| tasks.workflow.md | Workflow overview | References stage workflows |
| {stage}/README.md | Stage documentation | References parent and workflows |
| {stage}/INDEX.md | Stage task tracking | References stage README |
| {stage}/{stage}.workflow.md | Workflow definitions | Contains schemas & templates |
| llm.context.md | System architecture (this file) | Routes to everything |

---

## 5. Usage Patterns

### Creating a Task

1. Navigate to: `inbox/inbox.workflow.md`
2. Follow: **WF1 - Create Task Package**
3. Use templates provided in that workflow file

### Understanding Workflows

1. **Overview**: See `tasks.workflow.md` for all workflows
2. **Details**: See `{stage}/{stage}.workflow.md` for specific workflows
3. **Schemas**: Workflow files contain templates and schemas

### Finding Schemas

- **Task package schemas**: `README.md` (Minimal Schemas section)
- **Templates**: `inbox/inbox.workflow.md` (WF1 Templates section)
- **Event format**: `README.md` (Event Log Entry Format)
- **Index format**: `README.md` (Index File Format)

### Navigating Stages

Each stage folder is self-contained:
- Read `{stage}/README.md` for stage purpose and criteria
- Check `{stage}/INDEX.md` for current tasks
- Review `{stage}/{stage}.workflow.md` for workflows

### Following the Knowledge Graph

Documents have YAML frontmatter with:
- `type`: Document classification
- `parent`: Reference to parent document
- `stage`: Stage identifier (if applicable)
- `last_revisited` or `last_updated`: Maintenance dates

Follow parent references to navigate up the hierarchy.

---

## 6. Workflows Overview

The system has **9 primary workflows** across 7 stages:

| Workflow | Stage | Purpose | Details |
|----------|-------|---------|---------|
| WF1 | inbox | Create Task Package | See inbox/inbox.workflow.md |
| WF2 | inbox | Verify & Move to Backlog | See inbox/inbox.workflow.md |
| WF3 | backlog | Prioritize & Move to Next | See backlog/backlog.workflow.md |
| WF4 | next | Move to Current | See next/next.workflow.md |
| WF5 | current | Move to Done | See current/current.workflow.md |
| WF6 | integration | Integration & Technical Knowledge | See integration/integration.workflow.md |
| WF7 | learning | System-of-Work Review | See learning/learning.workflow.md |
| WF8 | learning | Learning Pattern Collection (periodic) | See learning/learning.workflow.md |
| WF9 | archived | Final Closure | See archived/archived.workflow.md |

**For complete workflow details**: See `tasks.workflow.md` and individual stage workflow files.

### Workflow Evolution

- Workflows reviewed after **25 executions** (configurable via frontmatter)
- Changes emerge from data and effectiveness signals
- Ineffective workflows are refined or removed
- New workflows created as needs emerge

### Workflow Criticality & Skip Philosophy

**Criticality Levels**:
- **flexible**: Trust-based - agents can adapt/skip with judgment
- **must**: Critical - skipping risks system integrity (security, data corruption, lost tasks)

**Within Workflows**:
- ⚠️ **MUST** steps: Protect system integrity (e.g., ledger logging, index updates, task movement)
- ✓ Flexible steps: Adapt as needed (e.g., verification depth, documentation detail)

**Skip Philosophy**:
- **Maximum Trust**: Agents trusted to make good decisions
- **Maximum Flexibility**: Can adapt workflows to be effective
- **Maximum Accountability**: All actions (including skips) logged
- **Effectiveness as Truth**: Skips are valuable signals, not failures

**Skip Signaling**:
- Every skip indicates: workflow may be obsolete, ineffective, or improvable
- Document skip reasons when helpful - benefits collective learning
- Not documenting still okay - trust over bureaucracy
- **Patterns emerge**: Multiple skips → workflow review trigger

**Principles**:
- Accept mistakes, assume good intent
- Never accept maliciousness, laziness, complacency
- Focus on execution excellence, not results excellence
- Control execution (we can), not results (we can't)
- Be principled, not rigid

---

## 7. Knowledge Graph Navigation

### Frontmatter Structure

Documents use minimal YAML frontmatter:

```yaml
---
type: workflow | stage_documentation | index | system_overview | workflow_overview | root_index
stage: inbox | backlog | next | current | integration | archived (if applicable)
parent: ../path/to/parent.md
revisit_after_executions: 25 (for workflows)
last_revisited: YYYY-MM-DD (for workflows)
last_updated: YYYY-MM-DD (for indexes and documentation)
---
```

### Navigation Patterns

**Top-Down**: Start at README.md → tasks.workflow.md → {stage}/README.md → {stage}/{stage}.workflow.md

**Bottom-Up**: Start at any file → follow `parent` reference → continue until root

**Lateral**: From stage README → sibling stage folders (via directory structure)

### Graph Properties

- **Acyclic**: No circular references
- **Rooted**: All paths lead to README.md
- **Hierarchical**: Clear parent/child relationships
- **Browseable**: Follow references without tools

---

## 8. System Maintenance

### When to Update This File

Update this llm.context.md when:
- ✅ System philosophy changes
- ✅ Core concepts evolve
- ✅ Architecture changes
- ✅ File structure pattern changes
- ✅ New system-level features added

Do NOT update when:
- ❌ Adding/removing workflows (they live in workflow files)
- ❌ Changing schemas (they live in README.md and workflow files)
- ❌ Adding/removing tasks (tracked in indexes)
- ❌ Updating stage criteria (tracked in stage READMEs)

### Workflow Maintenance

Workflows self-document their revisit schedule via frontmatter. After N executions, review:
- Is the workflow still effective?
- Should steps be added/removed/refined?
- Is the workflow still needed?

### Schema Evolution

Schemas live in:
- Task package structure: `README.md`
- Templates: `inbox/inbox.workflow.md`
- Event format: `README.md`

Update schemas there, not here.

---

## 9. Integration Points

### With Repository

This task management system is **independent** and can be:
- Used in multiple repositories
- Forked and adapted
- Evolved separately from code

### With Governance

Task management decisions that affect the entire repository should be captured in:
- `governance/DECISIONS.md` - High-level ADR about using this system
- `.work/tasks/` - System implementation details (here)

This separation keeps governance lean and task system detailed.

### With Code

Tasks track work on code but are separate:
- Tasks reference code locations in context files
- Task IDs can be referenced in commits
- Task ledgers can link to PRs/releases
- But task system doesn't require code to function

---

## 10. Routing Table (Where to Find What)

| Need | Location |
|------|----------|
| System overview & schemas | `README.md` |
| All workflows overview | `tasks.workflow.md` |
| Stage documentation | `{stage}/README.md` |
| Stage workflows | `{stage}/{stage}.workflow.md` |
| System workflows (WF0, content versioning) | `workflows/system.workflow.md` |
| State management workflows (block, pause, cancel, handoff, split) | `workflows/state-management.workflow.md` |
| Graph operations workflows (link, merge, validate) | `workflows/graph-operations.workflow.md` |
| Priority & effort workflows | `workflows/priority-effort.workflow.md` |
| Current task list (all) | `INDEX.md` |
| Current task list (stage) | `{stage}/INDEX.md` |
| Task package | `{stage}/TASK-{id}-{name}/` |
| System philosophy & architecture | This file (`llm.context.md`) |

---

## 11. Quick Reference

### File Extensions & Meanings

- `.md` - Markdown documentation
- `.jsonl` - JSON Lines (append-only logs)
- `.workflow.md` - Workflow definition files

### Common Patterns

**Creating**: Always start in inbox with WF1
**Moving**: Cut from source INDEX, paste to destination INDEX
**Logging**: Append to `{id}.ledger.jsonl`, never edit existing lines
**Referencing**: Use relative paths, follow parent links

### Key Invariants

- ✅ Task packages always have exactly 3 files
- ✅ Ledger is append-only, never edited
- ✅ One DRI per task, responsible end-to-end
- ✅ Human sign-off required at key stages
- ✅ Indexes always synchronized with task locations

---

## End of llm.context.md

This file documents the task management **system**, not the **implementation**. For workflows, schemas, and templates, see the referenced files above.

When in doubt, start at `README.md` and follow the routing table.
