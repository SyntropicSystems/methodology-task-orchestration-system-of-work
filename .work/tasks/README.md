---
type: system_overview
last_revisited: 2025-10-18
---

# Task Management System

## Philosophy

The purpose is task tracking in a dependency graph with full event log, provenance, and full trace graph.

We will extend, refine, delete workflows through emergence and signal intelligence, doing what serves us, removing what does not. Workflows shall have a review trigger that is not time based but after x executions. Effectiveness is the measurement of truth. 

We go from human → human-ai → ai-human → ai → automation by emergence. We automate what we can, but we invest smartly. Whatever we can do deterministically we will move down the chain to automate it as early as possible, while leaving bounded determinism to AI, and things that are one-offs or where humans are most capable of to humans.

Before any task is moved to next, integration, and archived, a human has to sign off. Each task needs one DRI that is responsible that the task is properly created, executed, implemented, integrated, and archived, end to end, while we also might have DRIs for the system itself. However the person owning the task is always responsible end to end that everything is done correctly. They do not need to be the creator, implementer, and so on, but they are responsible for the orchestration and results.

## Directory Structure

```
.work/tasks/
├── README.md              # This file - system overview
├── INDEX.md               # Root index of all tasks
├── inbox/                 # New tasks awaiting verification
│   ├── README.md
│   ├── INDEX.md
│   └── TASK-{id}-{name}/
├── backlog/               # Verified tasks awaiting prioritization
│   ├── README.md
│   ├── INDEX.md
│   └── TASK-{id}-{name}/
├── next/                  # Prioritized tasks ready to work on
│   ├── README.md
│   ├── INDEX.md
│   └── TASK-{id}-{name}/
├── current/               # Tasks actively being worked
│   ├── README.md
│   ├── INDEX.md
│   └── TASK-{id}-{name}/
├── integration/           # Completed tasks undergoing technical integration
│   ├── README.md
│   ├── INDEX.md
│   └── TASK-{id}-{name}/
├── learning/              # Tasks in system-of-work reflection (double-loop learning)
│   ├── README.md
│   ├── INDEX.md
│   └── TASK-{id}-{name}/
└── archived/              # Fully completed and closed tasks
    ├── README.md
    ├── INDEX.md
    └── TASK-{id}-{name}/
```

## Minimal Schemas

### Task ID Format

- **4-digit numeric**: 0001, 0002, ..., 9999
- **Repository-scoped**: One counter per repository (`.work/tasks/ID_COUNTER.txt`)
- **Monotonic**: IDs increment and are never reused
- **Reservation**: ID assigned during WF1 pre-step by reading and incrementing counter

### Task Package Structure

Each task is a folder containing exactly these files:

```
TASK-{id}-{name}/
├── {id}-{name}.task.md     # Task card (goal, inputs, outputs, acceptance)
├── {id}.context.md          # Context documentation (hermetic, self-contained)
├── {id}.ledger.jsonl        # Append-only event log
└── {id}.learning.md         # Learning document (created in learning stage, optional)
```

### Priority Levels (P0-P4)

| Level | Name | Meaning |
|-------|------|---------|
| P0 | Critical | System down, security breach (bypasses normal flow) |
| P1 | High | High-value, time-sensitive |
| P2 | Medium | Standard work (default) |
| P3 | Low | Nice-to-have |
| P4 | Defer | Acknowledged but not planned |

### Effort Estimates

- **Format**: XS | S | M | L | XL (or numeric: 1, 2, 3, 5, 8)
- **Required**: Before moving backlog → next
- **Purpose**: Capacity planning and split triggers

### Task Card Format

File: `{id}-{name}.task.md`

**YAML Frontmatter** (required):
```yaml
---
task_id: "0001"
created_by: "@user"
created_date: "YYYY-MM-DD"
schema_version: "1.0"
task_name: "Short name"
stage: "inbox"
priority: null  # P0|P1|P2|P3|P4
effort_estimate: null  # XS|S|M|L|XL
tags: []
current_dri: "@user"
status: "active"  # active|blocked|paused|canceled
blocked_by: null
paused_reason: null
depends_on: []
blocks: []
related_to: []
split_from: null
split_into: []
---
```

**Required Markdown Sections**:
- **Goal**: What needs to be accomplished
- **Inputs Required**: Dependencies, prerequisites, information needed
- **Implementation Details**: How to accomplish the goal
- **Expected Outputs**: Deliverables, artifacts, results
- **Acceptance Criteria**: Testable conditions for completion verification

### Event Log Entry Format

File: `{id}.ledger.jsonl`

Each line is a JSON object:

```json
{"ts":"2025-10-18T10:30:00Z","user":"user_id","event":"created","desc":"brief description"}
```

Fields:
- `ts`: ISO 8601 timestamp
- `user`: User identifier
- `event`: Event type (see vocabulary below)
- `desc`: Brief description of what happened

**Complete Event Vocabulary**:

*Core Lifecycle*:
- `id_reserved` - Task ID reserved
- `created` - Task package created
- `verified` - Task verified and moved to backlog
- `prioritized` - Task prioritized and moved to next
- `started` - Task moved to current, implementation started
- `implemented` - Implementation completed
- `integrated` - Task integrated and released
- `learning_documented` - Learning document created
- `closed` - Task closed and archived

*State Management*:
- `blocked` - Task blocked by dependency or external factor
- `unblocked` - Task unblocked, ready to proceed
- `paused` - Task temporarily suspended
- `resumed` - Task resumed from paused state
- `canceled` - Task abandoned/canceled

*Hand-off*:
- `handoff_initiated` - Hand-off process started
- `handoff_accepted` - New DRI accepted responsibility

*Graph Operations*:
- `split` - Task split into multiple tasks
- `merged_from` - Task absorbed another task
- `dependency_updated` - Dependency added or removed

*Failure Handling*:
- `rollback` - Task moved backward in lifecycle
- `rework_required` - Verification failed, needs rework

*Content & Metadata*:
- `content_updated` - Task card or context modified
- `priority_set` - Priority assigned
- `priority_changed` - Priority modified
- `effort_estimated` - Effort estimate assigned
- `tags_updated` - Tags added or removed

*System*:
- `index_synced` - Index consistency validated
- `index_drift_detected` - Index inconsistency found
- `move_blocked` - Move prevented due to validation failure

### Index File Format

File: `INDEX.md` (in each stage folder and root)

**Stage Index Format**:
```markdown
| ID | Name | DRI | Status | Flags | Dependencies |
|----|------|-----|--------|-------|--------------|
| 0001 | Example Task | @user | active | - | - |
| 0002 | Another Task | @user | active | blocked(0001) | TASK-0001 |
```

**Root Index Format** (adds Stage column):
```markdown
| ID | Name | DRI | Stage | Status | Flags | Dependencies |
|----|------|-----|-------|--------|-------|--------------|
| 0001 | Example Task | @user | inbox | active | - | - |
| 0002 | Another Task | @user | backlog | active | blocked(0001) | TASK-0001 |
```

Required columns:
- **ID**: Task identifier (4-digit number: 0001, 0002, ...)
- **Name**: Short task name
- **DRI**: Directly Responsible Individual
- **Stage**: Current stage (root index only)
- **Status**: Operational state (active|blocked|paused|canceled)
- **Flags**: State indicators (e.g., `blocked(TASK-0001)`, `paused(2025-11-01)`, `split(children)`)
- **Dependencies**: Task IDs this depends on (or `-` for none)

**Flags Examples**:
- `blocked(TASK-0001)` - Blocked by another task
- `blocked(external:API access)` - Blocked by external factor
- `paused(2025-11-01)` - Paused until date
- `split(children)` - Parent of split tasks
- `dep_ready` - All dependencies satisfied

## Workflows

Workflows are organized by stage. Each stage has its own workflow file containing detailed procedures.

### Workflow Overview

- **WF1** - Create Task Package (inbox)
- **WF2** - Verify Task Package and Move to Backlog (inbox)
- **WF3** - Prioritization and Moving Tasks to Next (backlog)
- **WF4** - Move to Current (next)
- **WF5** - Move to Done (current)
- **WF6** - Integration and Technical Knowledge Capture (integration)
- **WF7** - System-of-Work Review and Learning (learning)
- **WF8** - Learning Pattern Collection (learning - periodic)
- **WF9** - Final Closure (archived)

**For detailed workflow procedures, see**:
- Root workflow overview: [tasks.workflow.md](tasks.workflow.md)
- Stage-specific workflows: Each stage folder contains a `{stage}.workflow.md` file

## Learning Stage Policy

**Status**: FLEXIBLE (recommended but not strictly required)

**Purpose**: The learning stage provides space for double-loop learning - reflecting on the system-of-work itself, not just technical outcomes.

**Rules**:
- Archival does **not** strictly require passing through `learning` stage
- DRI must log one of:
  ```json
  {"ts":"ISO8601","user":"{DRI}","event":"learning_completed","desc":"See {id}.learning.md"}
  ```
  OR
  ```json
  {"ts":"ISO8601","user":"{DRI}","event":"learning_skipped","desc":"Reason: no meaningful process learnings / duplicate of TASK-XXXX"}
  ```
- **Sign-off**: Human sign-off is required at both `integration` and `archived` stages regardless of learning stage usage
- **Learning documents** (`{id}.learning.md`) capture process insights that improve how we work
- **Skip when**: Task followed standard patterns with no unique process insights

**Philosophy**: Learning is encouraged and valuable, but we trust DRIs to determine when it adds value versus creates overhead.

---

## Concurrency Coordination

**Context**: This system is designed for manual human execution before automation. Concurrent operations require social coordination protocols.

**Principles**:
- **Single Writer Rule**: Only one person modifies a task at a time
- **Announce and Complete**: Communicate intent before modifying shared state
- **Sequential Task Creation**: One person creates tasks at a time to avoid ID collisions
- **Workflow Atomicity**: Complete workflows in one sitting without interruption

**Coordination Protocols**:

### Before Modifying a Task
1. Announce in team chat: "Modifying TASK-0042"
2. Check no one else is working on it
3. Proceed with modification
4. Announce completion: "TASK-0042 modification complete"

### Before Creating Tasks
1. Announce: "Creating a task" (reserves ID_COUNTER access)
2. Execute WF1 completely
3. Announce: "Task created" (releases ID_COUNTER)

### If Workflow Interrupted
1. Log in task ledger: `{"ts":"ISO8601","user":"user_id","event":"workflow_interrupted","desc":"WF4 interrupted at step X, will retry"}`
2. Note the interruption point
3. On resume, check system state and restart or continue workflow

### If Conflict Detected
1. Compare both versions of modified files
2. Use ledger timestamps to determine chronological order
3. Merge changes manually if possible
4. Document conflict resolution in both task ledgers
5. If irreconcilable, use WF0 to detect and resolve drift

**When Tooling Arrives**: These protocols will be replaced by file locks, atomic transactions, optimistic concurrency, and automatic conflict detection.

---

## Governance Files

### Core System Files

| File | Purpose |
|------|---------|
| README.md | This file - system overview and schemas |
| INDEX.md | Root index of all tasks across all stages |
| tasks.workflow.md | Workflow overview with references to stage workflows |
| KNOWN_LIMITATIONS.md | Documented limitations and manual workarounds |

### Operating System Layer (Phase 2)

The Operating System Layer provides infrastructure for safe speed, observability, agent integration, and continuous improvement:

**Flow Control & Observability**:
- `wip.policy.md` - WIP limits, SLO targets, queue discipline, P0 andon rules
- `metrics.schema.md` - Event-sourced metrics (lead time, throughput, flow efficiency, etc.)
- `reports.spec.md` - Standard reports (Weekly Flow, Blockers, Rework, P0 Postmortem, Learning)

**Agent Architecture**:
- `query.spec.md` - Consistent data access interface (canonical queries, multiple interfaces)
- `worker.contract.md` - Agent interface specification (capabilities, idempotency, error contracts)

**Extensibility & Scale**:
- `events.schema.md` - Event versioning and hook points for custom logic
- `namespaces.md` - Multi-team scaling support with cross-namespace dependencies
- `console.view.md` - Operator views (Now, Flow, Blockers, Rework, Learning)
- `flywheel.md` - Automation promotion framework (Human → AI → Automated)

**Status**: Phase 2 complete (100%) - Full specification ready for implementation
