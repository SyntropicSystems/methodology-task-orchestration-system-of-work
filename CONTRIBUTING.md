# Contributing

Thanks for your interest in the Task Orchestration System! This project is a **living specification** of a task management methodology designed for progressive automation and AI collaboration.

## Overview

This is **not a traditional software project** - it's a documented system of work that can be:
- **Adopted** - Used in your own projects for task management
- **Adapted** - Forked and customized for your specific context
- **Evolved** - Improved through emergence and signal intelligence
- **Implemented** - Codified into tooling (SDK, service, CLI)

## How to Contribute

### 1. Adoption Reports

**Share your experience** using this system:
- Open an issue with label `adoption-report`
- Describe your context (team size, project type, adaptations made)
- Share what worked well and what didn't
- Include metrics if available (lead time, throughput, etc.)

These reports are invaluable for understanding effectiveness across contexts.

### 2. Methodology Improvements

**Propose enhancements** to the system itself:
- Open an issue with label `methodology-improvement`
- Describe the problem or inefficiency you've observed
- Propose a solution with rationale
- Reference which workflows or documents would change
- Include evidence from your usage (signal intelligence)

**Examples:**
- "WF4 step 3 is redundant - we skip it every time"
- "Add validation rule to prevent circular dependencies"
- "Split learning stage into process vs technical learning"

### 3. Documentation Clarifications

**Improve clarity** of existing documentation:
- Open an issue with label `documentation`
- Point to the confusing section
- Explain what was unclear
- Propose clearer wording or additional examples

### 4. Tool Implementations

**Build tooling** around the system:
- Automation scripts for workflows
- Context builders for AI agents
- Visualization tools for task graphs
- Metric calculators from event logs
- CLI utilities for common operations

Share implementations via:
- Links to your repositories
- Pull requests to add references in documentation
- Blog posts or tutorials

## Contribution Principles

### Emergence Over Planning

We don't have a rigid roadmap. Instead:
- ✅ Changes emerge from actual usage signals
- ✅ Ineffective workflows are refined or removed
- ✅ Effectiveness is the measurement of truth
- ✅ Review triggers are metrics and execution-based (after N uses), not time-based

### Evidence-Based Changes

Proposals should include:
- **Problem**: What inefficiency or issue did you observe?
- **Context**: In what scenario did this occur?
- **Data**: How many times? What was the impact?
- **Solution**: What change would address this?
- **Tradeoffs**: What might this break or complicate?

### Maintain System Integrity

Protect core invariants:
- ✅ Event logs remain append-only (never edited)
- ✅ Human sign-off at key stages (next, integration, archived)
- ✅ Single source of truth (no duplication)
- ✅ Task packages always have exactly 3 files
- ✅ DRI accountability end-to-end

## How to Adopt This System

### Quick Start (Single Person)

1. **Copy the structure**:
   ```bash
   mkdir -p .work/tasks/{inbox,backlog,next,current,integration,learning,archived}
   cp -r examples/.work/tasks/*.md .work/tasks/
   ```

2. **Create your first task**:
   - Follow [.work/tasks/inbox/inbox.workflow.md](./.work/tasks/inbox/inbox.workflow.md)
   - Execute WF1 to create a task package

3. **Work through stages**:
   - Use workflows as checklists
   - Log events in task ledgers
   - Keep indexes synchronized

4. **Learn and adapt**:
   - After ~10 tasks, review what works
   - Adapt workflows to your context
   - Document learnings

### Team Adoption

1. **Start small**: One person or sub-team pilots the system

2. **Define roles**:
   - Who assigns priorities?
   - Who can move tasks between stages?
   - How do DRI handoffs work?

3. **Establish rhythm**:
   - Daily: Check current stage, move completed tasks
   - Weekly: Prioritize backlog, review blockers
   - Monthly: Review learning stage, evolve workflows

4. **Tool progressively**:
   - Start manual (understand the system)
   - Add scripts for repetitive operations
   - Build automation as patterns emerge
   - Follow the flywheel (Human → AI → Automated)

5. **Customize for context**:
   - Adjust WIP limits for your capacity
   - Modify priority levels for your domain
   - Add custom event types for your needs
   - Create namespace structure for multiple teams

### What to Customize

You **should** adapt:
- ✅ Priority levels and meanings
- ✅ Effort estimates and thresholds
- ✅ WIP limits per stage
- ✅ Workflow review triggers (after N executions)
- ✅ Custom event types for your domain
- ✅ Stage names (if needed for your culture)

You **should preserve**:
- ✅ Event sourcing (append-only logs)
- ✅ Task package structure (3 files)
- ✅ Human sign-off gates
- ✅ Single source of truth
- ✅ DRI accountability

## Workflow Evolution Process

When you discover a workflow is ineffective:

1. **Document the signal**:
   - How many times was this workflow skipped?
   - What steps were consistently unnecessary?
   - What steps were missing?

2. **Propose a change**:
   - Update the workflow document
   - Add a note in the workflow's revision history
   - Log the change rationale

3. **Test the change**:
   - Use the modified workflow for N iterations
   - Track effectiveness compared to previous version

4. **Keep or revert**:
   - If better: Keep the change, update documentation
   - If worse: Revert, document why it didn't work

## Documentation Discipline

### File Organization

Follow the established patterns:
- **README.md** files - Overview and entry points
- **INDEX.md** files - Tables tracking items
- **{stage}.workflow.md** - Workflow definitions
- **llm.context.md** - System architecture for AI
- **{spec-name}.md** - Specifications (events, metrics, etc.)

### YAML Frontmatter

Documents use frontmatter for the knowledge graph:
```yaml
---
type: workflow | stage_documentation | index | system_overview
stage: inbox | backlog | next | current | integration | learning | archived
parent: ../path/to/parent.md
revisit_after_executions: 25
last_revisited: YYYY-MM-DD
---
```

### Relative Links

Always use relative links, not absolute paths:
- ✅ `[README.md](./README.md)`
- ✅ `[tasks](../.work/tasks/)`
- ❌ `/Users/you/project/.work/tasks/`
- ❌ `C:\projects\.work\tasks\`

### Single Source of Truth

Never duplicate information:
- ✅ Reference the canonical location
- ✅ Link to detailed documentation
- ❌ Copy/paste schemas across files
- ❌ Repeat workflow steps in multiple places

## Pull Request Guidelines

For changes to this repository:

### Documentation Changes
- Can be committed directly to `main` if non-breaking
- Use conventional commit format:
  - `docs: clarify WF3 priority assignment`
  - `docs: fix broken links in QUICKSTART`

### Methodology Changes
- Open an issue first for discussion
- Create a branch: `methodology/<short-desc>`
- Include rationale and evidence in PR description
- Update affected documentation and examples
- Add entry to relevant workflow revision history

### Tool Additions
- Create a branch: `tooling/<tool-name>`
- Include README with usage instructions
- Add tests or examples
- Document in main README under "Tooling"

## Communication

### GitHub Issues

Use labels to categorize:
- `adoption-report` - Experience using the system
- `methodology-improvement` - Proposed changes to workflows
- `documentation` - Clarifications or fixes
- `tooling` - Tool implementations
- `question` - General questions

### Discussions

For open-ended topics:
- Philosophy and principles
- Alternative approaches
- Implementation strategies
- Comparison with other systems

## Code of Conduct

### Principles

- **Assume good intent** - Everyone is trying to help
- **Be specific** - Provide context and evidence
- **Accept mistakes** - We're all learning
- **Reject complacency** - Always seek improvement
- **Focus on execution** - Control what we can control
- **Be principled, not rigid** - Adapt with purpose

### What We Don't Tolerate

- ❌ Malicious behavior or sabotage
- ❌ Laziness or negligence
- ❌ Personal attacks or harassment
- ❌ Bad faith arguments
- ❌ Ignoring system integrity invariants

## License

By contributing, you agree that your contributions are licensed under the [MIT License](./LICENSE).

## Getting Help

- **Understanding the system**: Read [.work/tasks/llm.context.md](./.work/tasks/llm.context.md)
- **Workflow questions**: See [.work/tasks/tasks.workflow.md](./.work/tasks/tasks.workflow.md)
- **Quick help**: Post in GitHub Discussions
- **Bugs in documentation**: Open an issue

## Thank You

This system exists because of people like you who:
- Try it and share learnings
- Propose improvements based on real usage
- Build tooling to make it easier
- Share it with others

Every adoption, every report, every improvement makes the system better for everyone.

---

*Remember: Effectiveness is the measurement of truth. If something doesn't work, that's valuable signal.*
