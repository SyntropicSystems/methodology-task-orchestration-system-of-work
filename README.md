# Task Orchestration System - FlowOS System of Work

An event-sourced task management methodology designed for progressive automation: human → human-AI → AI-human → AI → automated workflows.

## What Is This?

This repository contains a **living specification** of a task orchestration system - a documented methodology that a perfect agent (human or AI) could execute. It's similar to a living spec or agentic workflow demonstration, designed to make "doing the right things easy, and the wrong things hard" on our mission to make speed safe and scale fluid.

**This is not executable code.** It's a system of work that can be:
- Adopted and adapted for your own projects
- Used as-is for task management
- Evolved through emergence and signal intelligence
- Eventually codified into an SDK, service, or CLI

## Repository Structure

```
.
├── .work/                    # The system of work
│   ├── tasks/               # Task management system (core)
│   │   ├── README.md        # System overview & schemas
│   │   ├── INDEX.md         # All tasks across all stages
│   │   ├── tasks.workflow.md  # Workflow overview
│   │   ├── llm.context.md   # LLM context & architecture
│   │   ├── inbox/           # New tasks
│   │   ├── backlog/         # Verified tasks
│   │   ├── next/            # Prioritized tasks
│   │   ├── current/         # Active work
│   │   ├── integration/     # Technical integration
│   │   ├── learning/        # Process reflection
│   │   └── archived/        # Completed tasks
│   └── knowledge/           # Reference documentation
│
├── README.md                # This file
├── QUICKSTART.md           # Fast introduction to the system
├── CONTRIBUTING.md         # How to adopt & contribute
└── SECURITY.md             # Security & AI agent safety
```

## Quick Start

**New here?** Start with [QUICKSTART.md](./QUICKSTART.md) for a fast introduction.

**Want the full picture?** Read [.work/tasks/README.md](./.work/tasks/README.md) for system details.

**Ready to adopt?** See [CONTRIBUTING.md](./CONTRIBUTING.md) for adoption guidance.

## Core Concepts

### Task Lifecycle

Tasks flow through seven stages in a dependency graph:

```
inbox → backlog → next → current → integration → learning → archived
```

Each task is a folder containing:
- `{id}-{name}.task.md` - Goal, inputs, outputs, acceptance criteria
- `{id}.context.md` - Hermetic context for implementation
- `{id}.ledger.jsonl` - Append-only event log (full provenance)

### Event Sourcing

Every state change is recorded in an immutable event log, providing:
- ✅ Complete provenance and traceability
- ✅ Full audit trail for compliance
- ✅ Replay capability for analysis
- ✅ Signal intelligence for workflow evolution

### Progressive Automation

The system is designed to evolve from human execution → AI assistance → full automation:

1. **Human** - Manual execution, learning patterns
2. **Human-AI** - AI proposes, human approves
3. **AI-Human** - AI executes, human reviews
4. **AI + Spot** - AI autonomous with human spot-checks
5. **Automated** - Fully deterministic automation

We automate what we can deterministically, use AI for bounded determinism, and keep humans for high-value judgment.

## Philosophy

### Emergence Over Planning

We evolve workflows through signal intelligence - doing what serves us, removing what doesn't. Workflows have review triggers after N executions (not time-based). **Effectiveness is the measurement of truth.**

### Human Accountability

Before any task moves to next, integration, or archived stages, a human must sign off. Each task has one DRI (Directly Responsible Individual) accountable end-to-end for proper creation, execution, implementation, integration, and archival.

### Single Source of Truth

No duplication. Everything references its canonical location. The system maintains this discipline through:
- Event logs (append-only, never edited)
- Indexes (synchronized with task locations)
- Knowledge graph (via YAML frontmatter)

## Key Features

### For Teams
- **Dependency Management** - Tasks form a directed acyclic graph (DAG)
- **Work-in-Progress Limits** - Flow control and capacity planning
- **Double-Loop Learning** - Process reflection stage for methodology improvement
- **Metrics & Observability** - Event-sourced flow metrics (lead time, throughput, etc.)

### For AI Agents
- **Bounded Context** - Each component defines AI context limits
- **Worker Contracts** - Agent interface specification for safe automation
- **Query Interface** - Consistent data access for humans and agents
- **Idempotency** - Agents propose events, don't apply directly

### For Scale
- **Namespaces** - Multi-team support with cross-namespace dependencies
- **Automation Flywheel** - Framework for promoting workflows to automation
- **Event Hooks** - Extension points for custom logic
- **Console Views** - Operator situational awareness

## Documentation

### Essential Reading
- **[QUICKSTART.md](./QUICKSTART.md)** - Fast introduction to the system
- **[.work/tasks/README.md](./.work/tasks/README.md)** - System overview and schemas
- **[.work/tasks/llm.context.md](./.work/tasks/llm.context.md)** - Architecture and mental models
- **[.work/tasks/tasks.workflow.md](./.work/tasks/tasks.workflow.md)** - All workflows overview

### Operating System Layer (Phase 2)
The infrastructure for safe speed, observability, and agent integration:
- **Flow Control** - [wip.policy.md](./.work/tasks/wip.policy.md) (WIP limits, SLOs, queue discipline)
- **Observability** - [metrics.schema.md](./.work/tasks/metrics.schema.md), [reports.spec.md](./.work/tasks/reports.spec.md)
- **Agent Architecture** - [query.spec.md](./.work/tasks/query.spec.md), [worker.contract.md](./.work/tasks/worker.contract.md)
- **Extensibility** - [events.schema.md](./.work/tasks/events.schema.md), [namespaces.md](./.work/tasks/namespaces.md)
- **Automation** - [flywheel.md](./.work/tasks/flywheel.md) (promotion framework)

## Next Steps

The next phase is to **codify this system** by building an SDK, service, and CLI around it - making the right things easy and the wrong things hard.

## Contributing

We welcome:
- **Adoptions** - Use this system in your projects and share learnings
- **Adaptations** - Fork and customize for your context
- **Improvements** - Propose enhancements to the methodology
- **Implementations** - Build tooling around the system

See [CONTRIBUTING.md](./CONTRIBUTING.md) for details.

## Security

This system involves AI agents reading and executing workflows. See [SECURITY.md](./SECURITY.md) for:
- AI agent safety considerations
- Prompt injection mitigations
- Data handling guidelines
- Access control recommendations

## License

[MIT License](./LICENSE) - Use, adapt, and evolve freely.

## Status

- **Phase 1**: Core task management system ✅ (Complete)
- **Phase 2**: Operating system layer ✅ (Complete - specifications ready)
- **Phase 3**: SDK/Service/CLI implementation 🚧 (Upcoming)

---

*Making speed safe and scale fluid.*
