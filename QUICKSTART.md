# Quick Start Guide

Get up and running with the Task Orchestration System in 15 minutes.

## What Is This?

A **task management methodology** with:
- ✅ Event-sourced workflow (complete provenance)
- ✅ Dependency tracking (directed acyclic graph)
- ✅ Progressive automation (human → AI → automated)
- ✅ Double-loop learning (improve the system itself)

**Not code.** A documented system of work that can be adopted, adapted, and evolved.

## Core Idea in 30 Seconds

Tasks flow through seven stages like this:

```
inbox → backlog → next → current → integration → learning → archived
```

Each task is a **folder** with three files:
1. **Task card** - What needs to be done
2. **Context** - Everything needed to do it
3. **Event log** - Immutable history of what happened

Every state change is logged. Every dependency is tracked. Every decision is traceable.

## Quick Concepts

| Concept | What It Means |
|---------|---------------|
| **Task** | A folder with 3 files (card, context, ledger) |
| **Stage** | Phase in lifecycle (inbox, backlog, next, current, etc.) |
| **Workflow** | Defined procedure for moving tasks between stages |
| **Event** | Logged state change in the append-only ledger |
| **DRI** | Directly Responsible Individual (owner of a task) |
| **Index** | Markdown table tracking all tasks in a stage |

## Your First Task (5 Minutes)

### 1. Understand the Structure

```
.work/tasks/
├── inbox/           # New tasks start here
├── backlog/         # Verified, awaiting priority
├── next/            # Prioritized, ready to start
├── current/         # Active work
├── integration/     # Done, being integrated
├── learning/        # Reflecting on process
└── archived/        # Complete and closed
```

### 2. Read a Workflow

Open [.work/tasks/inbox/inbox.workflow.md](./.work/tasks/inbox/inbox.workflow.md) and find **WF1 - Create Task Package**.

This is a checklist. Follow it to create your first task.

### 3. Create a Task

Following WF1:

```bash
# 1. Read the ID counter
cat .work/tasks/ID_COUNTER.txt
# Let's say it shows: 1

# 2. Increment it
echo "2" > .work/tasks/ID_COUNTER.txt

# 3. Create task folder
mkdir -p .work/tasks/inbox/TASK-0001-my-first-task

# 4. Create the three required files
cd .work/tasks/inbox/TASK-0001-my-first-task
touch 0001-my-first-task.task.md
touch 0001.context.md
touch 0001.ledger.jsonl
```

### 4. Fill in the Task Card

Use the template from WF1 (in the workflow file). At minimum:

```yaml
---
task_id: "0001"
created_by: "@yourname"
created_date: "2025-10-20"
schema_version: "1.0"
task_name: "My First Task"
stage: "inbox"
priority: null
effort_estimate: null
tags: ["learning"]
current_dri: "@yourname"
status: "active"
---

## Goal
Learn how to create a task in this system.

## Inputs Required
- QUICKSTART.md documentation
- .work/tasks/inbox/inbox.workflow.md

## Implementation Details
1. Read WF1
2. Create task package
3. Fill in required sections
4. Log creation event

## Expected Outputs
- Complete task package in inbox/

## Acceptance Criteria
- [ ] Task folder exists with all 3 files
- [ ] Task card has all required sections
- [ ] Creation event logged in ledger
- [ ] Task appears in inbox INDEX.md
```

### 5. Log the Creation Event

In `0001.ledger.jsonl`:

```json
{"ts":"2025-10-20T12:00:00Z","user":"yourname","event":"id_reserved","desc":"Reserved ID 0001"}
{"ts":"2025-10-20T12:01:00Z","user":"yourname","event":"created","desc":"Created learning task"}
```

### 6. Update the Index

Add to `.work/tasks/inbox/INDEX.md`:

```markdown
| ID | Name | DRI | Status | Flags | Dependencies |
|----|------|-----|--------|-------|--------------|
| 0001 | My First Task | @yourname | active | - | - |
```

**Congratulations!** You've created your first task.

## What's Next?

### Learn the Workflows

Each stage has a workflow file:
- **WF1** - Create Task ([inbox.workflow.md](./.work/tasks/inbox/inbox.workflow.md))
- **WF2** - Verify & Move to Backlog ([inbox.workflow.md](./.work/tasks/inbox/inbox.workflow.md))
- **WF3** - Prioritize ([backlog.workflow.md](./.work/tasks/backlog/backlog.workflow.md))
- **WF4** - Start Work ([next.workflow.md](./.work/tasks/next/next.workflow.md))
- **WF5** - Complete ([current.workflow.md](./.work/tasks/current/current.workflow.md))
- **WF6** - Integrate ([integration.workflow.md](./.work/tasks/integration/integration.workflow.md))
- **WF7-8** - Learn ([learning.workflow.md](./.work/tasks/learning/learning.workflow.md))
- **WF9** - Archive ([archived.workflow.md](./.work/tasks/archived/archived.workflow.md))

### Move Your Task Forward

1. **Verify it** (WF2) - Check completeness, move to backlog
2. **Prioritize it** (WF3) - Assign priority (P0-P4), move to next
3. **Start it** (WF4) - Move to current, begin implementation
4. **Complete it** (WF5) - Verify acceptance criteria, move to integration
5. **Integrate it** (WF6) - Technical integration, document learnings
6. **Reflect** (WF7) - Process learnings (optional but recommended)
7. **Archive it** (WF9) - Final closure

Each workflow is a checklist in the respective stage's workflow file.

## Key Principles

### 1. Everything Is Logged

Every state change goes in the event log:

```json
{"ts":"ISO8601","user":"username","event":"event_type","desc":"what happened"}
```

This gives you:
- Complete audit trail
- Provenance for every decision
- Data for process improvement

### 2. Human Sign-Off Required

At critical gates:
- ✅ Moving to `next` (prioritization)
- ✅ Moving to `integration` (completion)
- ✅ Moving to `archived` (closure)

A human must approve. This is by design.

### 3. Single Source of Truth

- Event logs: Append-only, never edited
- Indexes: Reflect task locations accurately
- Task packages: One folder, three files
- No duplication across documents

### 4. Emergence Over Planning

Workflows evolve based on effectiveness:
- If a step is always skipped → consider removing it
- If validation keeps failing → consider strengthening it
- If a workflow is clunky → propose improvements

**Effectiveness is the measurement of truth.**

## Common Operations

### Check Current Work

```bash
# See what's in each stage
cat .work/tasks/current/INDEX.md
cat .work/tasks/next/INDEX.md
```

### Find a Task

```bash
# By ID
cat .work/tasks/INDEX.md | grep "0001"

# By name
cat .work/tasks/INDEX.md | grep -i "first task"
```

### View Task History

```bash
cat .work/tasks/inbox/TASK-0001-my-first-task/0001.ledger.jsonl
```

Each line shows a state change with timestamp and user.

### See All Workflows

```bash
# Overview
cat .work/tasks/tasks.workflow.md

# Specific stage
cat .work/tasks/inbox/inbox.workflow.md
```

## Visual Overview

```
Task Lifecycle:
┌─────────┐     ┌─────────┐     ┌──────┐     ┌─────────┐
│  inbox  │────>│ backlog │────>│ next │────>│ current │
│  (new)  │     │(verify) │     │(prio)│     │ (work)  │
└─────────┘     └─────────┘     └──────┘     └─────────┘
                                                    │
                                                    v
┌─────────┐     ┌──────────┐     ┌─────────────┐
│archived │<────│ learning │<────│ integration │
│ (done)  │     │(reflect) │     │  (release)  │
└─────────┘     └──────────┘     └─────────────┘

Task Package:
TASK-0001-name/
├── 0001-name.task.md      # Goal, inputs, outputs, acceptance
├── 0001.context.md         # Everything needed to implement
└── 0001.ledger.jsonl       # Immutable event log

Event Log:
{"ts":"2025-10-20T12:00:00Z","user":"alice","event":"created","desc":"..."}
{"ts":"2025-10-20T12:05:00Z","user":"alice","event":"verified","desc":"..."}
{"ts":"2025-10-20T13:00:00Z","user":"bob","event":"prioritized","desc":"..."}
```

## Progressive Automation

The system is designed to evolve:

1. **Human** (start here) - Execute workflows manually
2. **Human-AI** - AI proposes, you approve
3. **AI-Human** - AI executes, you review
4. **AI + Spot** - AI autonomous, you spot-check
5. **Automated** - Fully deterministic

Start by executing workflows yourself. Learn the patterns. Then introduce AI assistance. Eventually automate what's deterministic.

## When Things Go Wrong

### Task in Wrong Stage?

1. Check the event log - what happened?
2. Check the workflow criteria - did you skip steps?
3. Move it back if needed (log a `rollback` event)

### Index Out of Sync?

1. Compare index to actual task folders
2. Use WF0 (in [workflows/system.workflow.md](./.work/tasks/workflows/system.workflow.md)) to detect drift
3. Correct the index, log an `index_synced` event

### Forgot to Log an Event?

Append it now (event logs are append-only, never edited):

```json
{"ts":"2025-10-20T14:00:00Z","user":"yourname","event":"late_log","desc":"Forgot to log earlier action"}
```

Honesty > perfection. The system learns from reality.

## Team Usage

### Coordination (Before Tooling)

1. **Announce** before modifying shared state
2. **One writer** per task at a time
3. **Log interruptions** if you must pause
4. **Document conflicts** if they occur

See [.work/tasks/README.md#concurrency-coordination](./.work/tasks/README.md#concurrency-coordination)

### Roles

- **DRI** - Owns the task end-to-end
- **Implementer** - Does the work (may be same as DRI)
- **Reviewer** - Approves stage transitions
- **System DRI** - Maintains the system itself

## Learn More

### Essential Reading

1. **This file** - You're here ✓
2. **[.work/tasks/README.md](./.work/tasks/README.md)** - System overview & schemas
3. **[.work/tasks/llm.context.md](./.work/tasks/llm.context.md)** - Architecture & mental models
4. **[.work/tasks/tasks.workflow.md](./.work/tasks/tasks.workflow.md)** - All workflows

### Deep Dives

- **Event Sourcing**: [.work/tasks/events.schema.md](./.work/tasks/events.schema.md)
- **Metrics**: [.work/tasks/metrics.schema.md](./.work/tasks/metrics.schema.md)
- **WIP Limits**: [.work/tasks/wip.policy.md](./.work/tasks/wip.policy.md)
- **AI Agents**: [.work/tasks/worker.contract.md](./.work/tasks/worker.contract.md)
- **Automation**: [.work/tasks/flywheel.md](./.work/tasks/flywheel.md)

### Contributing

Want to adopt this system? Have improvements? See [CONTRIBUTING.md](./CONTRIBUTING.md).

### Security

Using AI agents? Read [SECURITY.md](./SECURITY.md) for safety guidance.

## Quick Reference Card

```
Create Task:     WF1 (inbox)
Verify:          WF2 (inbox → backlog)
Prioritize:      WF3 (backlog → next)
Start:           WF4 (next → current)
Complete:        WF5 (current → integration)
Integrate:       WF6 (integration → learning)
Learn:           WF7 (learning → archived)
Archive:         WF9 (archived)

Task Package:    3 files (task.md, context.md, ledger.jsonl)
Event Format:    {"ts":"ISO8601","user":"name","event":"type","desc":"what"}
Index Format:    Markdown table with ID, Name, DRI, Status, Flags, Dependencies

Required Gates:  Human sign-off at next, integration, archived
Core Invariant:  Event logs are append-only (never edited)
Philosophy:      Effectiveness is the measurement of truth
```

## FAQ

**Q: Can I skip the learning stage?**  
A: Yes, it's flexible. Just log either `learning_completed` or `learning_skipped` before archival.

**Q: What if I make a mistake?**  
A: Log it. Append a correction event. The system learns from reality, not perfection.

**Q: Can I customize workflows?**  
A: Absolutely. Adapt to your context. Just preserve the core invariants (event sourcing, human sign-off, single source of truth).

**Q: How do I know if it's working?**  
A: Track metrics (lead time, throughput). Are tasks flowing? Are blockers resolved? Is the team learning?

**Q: What if my team doesn't follow the workflows?**  
A: That's valuable signal. Which steps are skipped? Why? Either enforce them (if critical) or remove them (if ineffective).

**Q: Can I use this with [tool/framework]?**  
A: Yes. This is methodology, not tooling. Use whatever tools work for you.

---

**Ready to dive deeper?** Start with [.work/tasks/README.md](./.work/tasks/README.md) for the complete system documentation.

**Questions?** Open an issue or check [CONTRIBUTING.md](./CONTRIBUTING.md) for how to get help.

*Now go create your second task. Then your third. The system reveals itself through usage.*
