---
type: specification
category: automation
last_updated: 2025-10-18
---

# Flywheel - Automation Promotion

This specification defines the automation flywheel: a systematic approach to evolving from human execution → human-AI collaboration → AI execution → full automation. Tasks progress through the flywheel as they demonstrate readiness for increased automation.

## Principles

1. **Progressive Automation**: Human → human-ai → ai-human → ai → automation
2. **Evidence-Based**: Promotion requires proven stability (metrics + passes)
3. **Reversible**: Can always demote back to more human control
4. **Transparent**: Clear criteria for each promotion level
5. **Human-in-Loop**: Critical approval gates remain human

## Automation Maturity Levels

### Level 0: Human Execution (Default)

**Who Executes**: Human DRI performs all steps manually  
**Use Case**: New task types, one-offs, complex judgment required

**Characteristics**:
- Human reads task card
- Human creates plan mentally or on paper
- Human implements solution
- Human validates work
- Human moves task through stages

**Example**: Novel architecture decision, customer escalation, strategic planning

---

### Level 1: Human-AI Collaboration

**Who Executes**: Human leads, AI assists with tools  
**Use Case**: Standard tasks with AI tool support

**Characteristics**:
- Human reads task, AI can fetch context
- Human creates plan, AI can suggest steps
- Human implements with AI code generation
- Human validates, AI can run tests
- Human moves task, AI validates prerequisites

**Promotion Criteria**:
- Task type repeats ≥3 times
- Clear acceptance criteria exist
- Predictable workflow pattern

**Example**: Feature implementation, bug fixes, documentation updates

---

### Level 2: AI-Human Review

**Who Executes**: AI executes, human reviews and approves  
**Use Case**: Routine tasks with human oversight

**Characteristics**:
- AI reads task and creates execution plan
- **Human approves plan** before execution
- AI implements solution using tools
- AI proposes completion state
- **Human reviews artifacts** and approves
- Human writes events to ledger after approval

**Promotion Criteria**:
- ≥5 successful executions at Level 1
- Rework rate <10% (low error rate)
- Average cycle time stable (±20%)
- All tasks pass WF-VALIDATE-MOVE without failures

**Example**: Standard feature additions, test writing, configuration updates

---

### Level 3: AI Execution with Spot Checks

**Who Executes**: AI executes autonomously, human spot-checks  
**Use Case**: Mature, stable task types

**Characteristics**:
- AI creates plan autonomously (no approval)
- AI implements solution
- AI validates with automated tests
- AI proposes move to next stage
- **Human spot-checks** 10-20% of completed tasks
- Human approval required for stage transitions

**Promotion Criteria**:
- ≥10 successful executions at Level 2
- Rework rate <5%
- Cycle time variance <15% (highly predictable)
- No SLO violations in last 10 tasks
- Automated test coverage ≥80%

**Example**: Standard bug fixes, dependency updates, routine refactoring

---

### Level 4: Full Automation

**Who Executes**: Fully automated pipeline  
**Use Case**: Commodity operations

**Characteristics**:
- Task created automatically (trigger or schedule)
- Automated planning and execution
- Automated validation and testing
- Automated stage transitions
- **Human notified** of completion (optional review)
- Human can intervene if anomalies detected

**Promotion Criteria**:
- ≥20 successful executions at Level 3
- Rework rate <2%
- Zero SLO violations in last 20 tasks
- Fully automated rollback capability
- Monitoring and alerting in place

**Example**: Dependency version bumps, certificate renewals, scheduled backups

---

## Flywheel Loop (7-Stage Process)

### Stage 1: Capture (Repeatability Detection)

**Purpose**: Identify task types that repeat frequently

**Process**:
1. Track task types by tags or name patterns
2. Count frequency of each task type
3. Flag types with ≥3 occurrences

**Output**: List of candidate task types for automation

**Example**:
```
Task Type: "Update dependencies"
Occurrences: 8 times in last 30 days
Tags: [dependency, maintenance]
→ CANDIDATE for automation
```

**Trigger**: Run monthly or when task archived

---

### Stage 2: Template (Standardization)

**Purpose**: Create reusable template for task type

**Process**:
1. Analyze past tasks of this type
2. Extract common patterns:
   - Goal structure
   - Input requirements
   - Acceptance criteria
   - Implementation steps
3. Create task template with placeholders

**Output**: Task template file

**Example Template**:
```yaml
# Template: dependency-update.template.yaml
task_type: "dependency_update"
goal: "Update {package} from {old_version} to {new_version}"
inputs_required:
  - package_name
  - current_version
  - target_version
  - repository_url
acceptance_criteria:
  - "Tests pass after update"
  - "No breaking changes introduced"
  - "CHANGELOG updated"
implementation_steps:
  - "Update package.json / requirements.txt"
  - "Run tests"
  - "Update CHANGELOG"
  - "Create pull request"
estimated_effort: "XS"
automated_at_level: 1  # Human-AI collaboration
```

---

### Stage 3: Validate (Quality Assurance)

**Purpose**: Ensure template matches actual task execution

**Process**:
1. Create 3-5 tasks from template
2. Execute tasks manually (Level 0 or Level 1)
3. Compare actual execution to template predictions
4. Measure deviations:
   - Did steps match template?
   - Was effort estimate accurate?
   - Did acceptance criteria cover all cases?

**Output**: Validated template or revision requirements

**Success Criteria**:
- ≥80% alignment between template and actual execution
- Effort estimates within ±50% of actual
- Zero acceptance criteria gaps

---

### Stage 4: Delegate (AI Worker Assignment)

**Purpose**: Assign AI worker to execute templated tasks

**Process**:
1. Register worker for this task type
2. Configure worker with:
   - Task template
   - Required tools and capabilities
   - Validation rules
   - Error handling procedures
3. Set automation level (start at Level 1)

**Output**: Worker assignment in worker registry

**Example**:
```yaml
# .work/tasks/.workers/dependency-updater.yaml
worker_id: "dependency-updater"
task_types: ["dependency_update"]
automation_level: 1  # Human-AI collaboration
capabilities:
  - plan
  - execute
  - validate
template: "dependency-update.template.yaml"
```

---

### Stage 5: Instrument (Metrics Collection)

**Purpose**: Collect execution metrics to enable learning

**Process**:
1. Add instrumentation hooks:
   - Execution duration per step
   - Test pass/fail rates
   - Rework frequency
   - Human intervention count
2. Log metrics to ledger or separate metrics file
3. Enable dashboard tracking

**Output**: Instrumented worker + metrics dashboard

**Metrics Collected**:
- Success rate (% of tasks completed without rework)
- Cycle time (start to completion)
- Rework rate (% requiring human correction)
- SLO compliance (% meeting time targets)
- Human intervention rate (% requiring human help)

---

### Stage 6: Learn (Pattern Recognition)

**Purpose**: Analyze metrics to determine readiness for promotion

**Process**:
1. Analyze collected metrics (after ≥5 executions)
2. Check promotion criteria for next level
3. Identify failure patterns if not ready
4. Generate promotion report

**Output**: Promotion recommendation or improvement actions

**Promotion Report Example**:
```yaml
task_type: "dependency_update"
current_level: 1  # Human-AI collaboration
executions: 12
metrics:
  success_rate: 91.7%  (11 of 12)
  avg_cycle_time: 2.3 hours
  cycle_time_variance: 18%
  rework_rate: 8.3%  (1 of 12)
  slo_compliance: 100%
  
promotion_evaluation:
  target_level: 2  # AI-Human review
  criteria:
    - executions_required: 5  ✓ (12 ≥ 5)
    - rework_rate: <10%  ✓ (8.3%)
    - cycle_time_stable: ±20%  ✓ (18%)
    - validation_passes: 100%  ✓
  
  recommendation: PROMOTE to Level 2
  confidence: HIGH
```

---

### Stage 7: Automate (Promotion or Demotion)

**Purpose**: Adjust automation level based on evidence

**Process**:
1. Human reviews promotion report
2. If approved, update worker automation level
3. If not ready, implement improvement actions
4. If performance degrades, demote to lower level

**Output**: Updated worker configuration

**Promotion**:
```yaml
# Update worker config
automation_level: 2  # Promoted to AI-Human review
promoted_date: "2025-10-20"
promotion_reason: "12 successful passes, 8.3% rework rate"
```

**Demotion** (if needed):
```yaml
# Demote if performance degrades
automation_level: 1  # Demoted back to Human-AI
demotion_date: "2025-11-05"
demotion_reason: "Rework rate increased to 25%, 3 SLO violations"
```

---

## Promotion Decision Matrix

| Current Level | Executions Required | Success Rate | Rework Rate | Cycle Time Variance | Next Level |
|---------------|---------------------|--------------|-------------|---------------------|------------|
| 0 → 1 | 3 | N/A | N/A | N/A | Human-AI |
| 1 → 2 | 5 | ≥90% | <10% | <20% | AI-Human |
| 2 → 3 | 10 | ≥95% | <5% | <15% | AI + Spot Check |
| 3 → 4 | 20 | ≥98% | <2% | <10% | Full Automation |

**Additional Requirements**:
- All levels: No security incidents, no data loss
- Level 3+: Automated tests with ≥80% coverage
- Level 4: Automated rollback + monitoring/alerting

## Worker Adapter with Predefined Steps

### Example: Dependency Update Worker

**Predefined Plan**:
```python
class DependencyUpdateWorker:
    """Level 2: AI-Human review worker for dependency updates."""
    
    def plan(self, task: Task) -> Plan:
        """Generate standardized plan from template."""
        return Plan(
            steps=[
                Step("Identify current version", duration="5m"),
                Step("Check for updates and breaking changes", duration="10m"),
                Step("Update package file", duration="5m"),
                Step("Run test suite", duration="15m"),
                Step("Update CHANGELOG", duration="5m"),
                Step("Create pull request", duration="5m"),
            ],
            estimated_effort="XS",
            total_duration="45m"
        )
    
    def validate(self, task: Task) -> ValidationResult:
        """Predefined validation checks."""
        checks = [
            self.check_tests_passing(),
            self.check_no_breaking_changes(),
            self.check_changelog_updated(),
        ]
        return ValidationResult(checks)
    
    def execute(self, task: Task, plan: Plan) -> ExecutionResult:
        """Execute with predefined steps."""
        for step in plan.steps:
            result = self.execute_step(step)
            if result.failed:
                return ExecutionResult(
                    success=False,
                    error=result.error,
                    proposed_events=[{
                        "event": "implementation_failed",
                        "desc": f"Step '{step.name}' failed: {result.error}"
                    }]
                )
        
        return ExecutionResult(
            success=True,
            artifacts=self.get_artifacts(),
            proposed_events=[{
                "event": "implemented",
                "desc": "Dependency updated successfully"
            }]
        )
```

---

## Human Role Evolution

### As Automation Increases

**Level 0-1**: Hands-on execution  
**Level 2**: Planning approval + artifact review  
**Level 3**: Spot-check quality + exception handling  
**Level 4**: System monitoring + policy adjustment

### Human Responsibilities by Level

| Level | Planning | Execution | Review | Approval |
|-------|----------|-----------|--------|----------|
| 0 | 100% human | 100% human | 100% human | 100% human |
| 1 | Human leads | 50/50 | 100% human | 100% human |
| 2 | AI proposes | 90% AI | 100% human | 100% human |
| 3 | 100% AI | 100% AI | 10-20% spot check | Stage moves only |
| 4 | 100% AI | 100% AI | Exception review | None (notified) |

**Humans move upstream to**:
- Designing new templates
- Reviewing promotion reports
- Handling exceptions and edge cases
- Improving worker capabilities
- Strategic planning and architecture

---

## Flywheel Dashboard

### Automation Maturity View

```
╔══════════════════════════════════════════════════════════════════╗
║                   AUTOMATION FLYWHEEL                             ║
║                  Last Updated: 2025-10-18                        ║
╠══════════════════════════════════════════════════════════════════╣
║ 📊 TASK TYPES BY AUTOMATION LEVEL                               ║
╠══════════════════════════════════════════════════════════════════╣
║ Level 0 (Human):          15 task types  ████████████████░░░░░░  ║
║ Level 1 (Human-AI):        8 task types  ████████░░░░░░░░░░░░░░  ║
║ Level 2 (AI-Human):        5 task types  █████░░░░░░░░░░░░░░░░░  ║
║ Level 3 (AI + Spot):       2 task types  ██░░░░░░░░░░░░░░░░░░░░  ║
║ Level 4 (Automated):       1 task type   █░░░░░░░░░░░░░░░░░░░░░  ║
╠══════════════════════════════════════════════════════════════════╣
║ 🎯 PROMOTION CANDIDATES                                          ║
╠══════════════════════════════════════════════════════════════════╣
║ Task Type              │ Current │ Ready For │ Executions │ Conf ║
║────────────────────────┼─────────┼───────────┼────────────┼──────║
║ dependency_update      │ L1      │ L2        │ 12         │ HIGH ║
║ bug_fix_minor          │ L2      │ L3        │ 15         │ MED  ║
║ test_addition          │ L1      │ L2        │ 8          │ LOW  ║
╠══════════════════════════════════════════════════════════════════╣
║ ⚠️  AUTOMATION HEALTH                                            ║
╠══════════════════════════════════════════════════════════════════╣
║ • dependency_update: Stable (8.3% rework, ready for L2)         ║
║ • bug_fix_minor: Excellent (2% rework, ready for L3)            ║
║ • test_addition: Needs more data (only 8 executions)            ║
║                                                                   ║
║ 🚀 Automation Coverage: 38% of tasks use Level 2+ automation    ║
╚══════════════════════════════════════════════════════════════════╝
```

---

## Implementation Roadmap

### Phase 1: Foundation (Weeks 1-4)

- Implement Level 0 (human execution) - already done
- Implement Level 1 (human-AI collaboration)
- Create first 3 task templates
- Set up metrics collection

### Phase 2: Templating (Weeks 5-8)

- Analyze top 10 task types
- Create templates for repeating tasks
- Validate templates with human execution
- Configure workers for Level 1

### Phase 3: Delegation (Weeks 9-12)

- Promote 3-5 task types to Level 2
- Collect execution metrics
- Build flywheel dashboard
- Generate first promotion reports

### Phase 4: Optimization (Weeks 13+)

- Promote high-performing task types to Level 3
- Automate Level 4 candidates (1-2 types)
- Continuous monitoring and adjustment
- Expand to more task types

---

## Demotion and Rollback

### When to Demote

**Triggers**:
- Rework rate exceeds threshold for level
- Multiple SLO violations (≥3 in last 10 tasks)
- Security incident or data loss
- Significant process changes invalidate template

**Process**:
1. Detect performance degradation via metrics
2. Alert human operators
3. Review recent failures
4. Decide: fix worker or demote
5. If demote: update worker config, notify team

**Example**:
```
Task Type: dependency_update
Current Level: 2 (AI-Human review)
Issue: Rework rate increased from 8% to 28% (last 5 tasks)
Root Cause: Breaking changes in major version updates not detected
Action: DEMOTE to Level 1 (Human-AI) until detection improved
```

---

## References

- Worker contract: `worker.contract.md`
- Metrics schema: `metrics.schema.md`
- Query interface: `query.spec.md`
- Reports specification: `reports.spec.md`
- Event schema: `events.schema.md`
- Workflow integration: `tasks.workflow.md`
