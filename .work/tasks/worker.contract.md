---
type: contract
category: agent_interface
last_updated: 2025-10-18
---

# Worker Contract - Agent Interface

This contract defines the interface between the task management system and worker agents (AI assistants, automation scripts, humans). It makes the system truly agent-agnostic by specifying standard capabilities, contracts, and protocols.

## Principles

1. **Capability-Based**: Workers declare what they can do (plan, validate, execute, etc.)
2. **Idempotent**: Workers propose changes, don't apply them directly (approval workflow)
3. **Hermetic**: Each task package is self-contained (no external context required)
4. **Observable**: All worker actions logged in task ledgers
5. **Fallback-Safe**: Human can always intervene or override

## Worker Types

### Type 1: Planning Worker
**Capabilities**: `plan`, `decompose`, `estimate`  
**Use Case**: Break down complex tasks, create subtasks, estimate effort

### Type 2: Validation Worker
**Capabilities**: `validate`, `verify`, `check_dependencies`  
**Use Case**: Pre-flight checks, dependency validation, acceptance criteria verification

### Type 3: Execution Worker
**Capabilities**: `execute`, `implement`, `test`  
**Use Case**: Code generation, implementation, testing

### Type 4: Review Worker
**Capabilities**: `review`, `suggest_improvements`, `identify_risks`  
**Use Case**: Code review, quality checks, risk analysis

### Type 5: Integration Worker
**Capabilities**: `merge`, `deploy`, `verify_integration`  
**Use Case**: Merge code, deploy changes, verify integration

### Type 6: Learning Worker
**Capabilities**: `extract_patterns`, `document_learnings`, `suggest_improvements`  
**Use Case**: Extract patterns from completed tasks, suggest process improvements

## Worker Capability Manifest

Each worker must provide a capability manifest declaring what it can do.

**Manifest Format** (`.work/tasks/.workers/{worker_id}.manifest.yaml`):
```yaml
worker_id: "claude-3.5-sonnet"
worker_type: "execution_worker"
version: "1.0"
capabilities:
  - plan
  - validate
  - execute
  - review
  - summarize
  - propose_move
tools:
  - read_file
  - write_file
  - execute_command
  - search_files
limits:
  max_file_size_mb: 10
  max_files_per_task: 50
  max_execution_time_minutes: 30
  sandbox: true
error_types:
  - validation_failed
  - dependency_unmet
  - test_failed
  - timeout
  - resource_limit_exceeded
```

## Core Capabilities

### 1. Capability: `plan`

**Purpose**: Create execution plan for a task  
**Input**: Task package (task.md, context.md, ledger.jsonl)  
**Output**: Proposed plan with steps and estimated effort

**Interface**:
```python
def plan(task_package: TaskPackage) -> Plan:
    """Create execution plan for task."""
    pass
```

**Output Schema**:
```yaml
plan:
  task_id: "0042"
  steps:
    - step: 1
      action: "Read authentication requirements from context"
      estimated_time: "5 minutes"
    - step: 2
      action: "Implement OAuth2 flow"
      estimated_time: "2 hours"
    - step: 3
      action: "Write tests"
      estimated_time: "1 hour"
    - step: 4
      action: "Update documentation"
      estimated_time: "30 minutes"
  total_estimated_time: "3.6 hours"
  estimated_effort: "M"
  risks:
    - "OAuth2 library may have breaking changes"
    - "Integration testing requires test credentials"
  dependencies_needed:
    - task_id: "0010"
      reason: "API credentials required"
```

**Worker Responsibility**:
- Read task card (goal, inputs, expected outputs, acceptance criteria)
- Read context document for additional information
- Break down into concrete steps
- Estimate time/effort realistically
- Identify risks and dependencies
- **Propose** plan, don't execute yet

---

### 2. Capability: `validate`

**Purpose**: Validate task package or proposed state change  
**Input**: Task package or proposed event  
**Output**: Validation result (pass/fail with reasons)

**Interface**:
```python
def validate(task_package: TaskPackage, proposed_action: str) -> ValidationResult:
    """Validate task readiness or proposed action."""
    pass
```

**Validation Checks**:
- Package completeness (all 3 files present)
- Frontmatter schema compliance
- Acceptance criteria clarity
- Dependency availability (all depends_on tasks exist and are ready)
- No circular dependencies
- Stage transition rules (e.g., can't move to current if dependencies blocked)
- WIP limits not exceeded

**Output Schema**:
```yaml
validation:
  task_id: "0042"
  action: "move_to_current"
  result: "fail"
  checks:
    - check: "package_complete"
      status: "pass"
    - check: "dependencies_ready"
      status: "fail"
      reason: "TASK-0010 is blocked"
    - check: "wip_limit"
      status: "pass"
  recommendation: "Wait for TASK-0010 to unblock before moving to current"
```

**Worker Responsibility**:
- Run all validation checks from `WF-VALIDATE-MOVE` (see `workflows/validation.workflow.md`)
- Return clear pass/fail with specific reasons
- Suggest remediation if validation fails

---

### 3. Capability: `execute`

**Purpose**: Implement the task  
**Input**: Task package + execution plan  
**Output**: Implementation artifacts + summary

**Interface**:
```python
def execute(task_package: TaskPackage, plan: Plan) -> ExecutionResult:
    """Execute task implementation."""
    pass
```

**Worker Responsibility**:
- Follow the approved plan
- Create/modify files as needed
- Run tests
- Document changes
- Log progress events
- **Propose** completed state, wait for human verification

**Output Schema**:
```yaml
execution:
  task_id: "0042"
  status: "completed"
  artifacts:
    - path: "src/auth/oauth2.py"
      change_type: "created"
      lines_added: 250
    - path: "tests/test_auth.py"
      change_type: "created"
      lines_added: 120
    - path: "README.md"
      change_type: "modified"
      lines_changed: 15
  tests_passed: true
  proposed_events:
    - event: "implemented"
      desc: "OAuth2 flow implemented with tests"
  summary: |
    Implemented OAuth2 authentication flow with:
    - Authorization code flow
    - Token refresh mechanism
    - Session management
    - 95% test coverage
```

**Idempotency Rule**:
- Worker **proposes** events, doesn't write to ledger directly
- Human reviews execution result and approves/rejects
- Only after approval, events are written to ledger

---

### 4. Capability: `review`

**Purpose**: Review completed work  
**Input**: Task package + implementation artifacts  
**Output**: Review feedback (approve/request changes)

**Interface**:
```python
def review(task_package: TaskPackage, artifacts: List[Artifact]) -> ReviewResult:
    """Review completed implementation."""
    pass
```

**Review Criteria**:
- Acceptance criteria met
- Code quality (readability, maintainability)
- Test coverage adequate
- Documentation updated
- No obvious bugs or security issues

**Output Schema**:
```yaml
review:
  task_id: "0042"
  reviewer: "claude-3.5-sonnet"
  result: "approve_with_suggestions"
  acceptance_criteria:
    - criterion: "OAuth2 flow implemented"
      met: true
    - criterion: "Token refresh working"
      met: true
    - criterion: "Tests pass"
      met: true
  suggestions:
    - "Add rate limiting to token refresh"
    - "Consider adding refresh token rotation"
  issues: []
  recommendation: "Approve for integration"
```

---

### 5. Capability: `summarize`

**Purpose**: Generate concise task summary  
**Input**: Task package  
**Output**: Summary for reports/dashboards

**Interface**:
```python
def summarize(task_package: TaskPackage) -> Summary:
    """Generate task summary."""
    pass
```

**Output Schema**:
```yaml
summary:
  task_id: "0042"
  one_line: "Implemented OAuth2 authentication flow"
  status: "completed"
  key_points:
    - "OAuth2 authorization code flow"
    - "Token refresh mechanism"
    - "95% test coverage"
  blockers: []
  next_action: "Ready for integration"
```

---

### 6. Capability: `propose_move`

**Purpose**: Propose moving task to next stage  
**Input**: Task package + target stage  
**Output**: Proposed move with validation

**Interface**:
```python
def propose_move(task_package: TaskPackage, target_stage: str) -> MoveProposal:
    """Propose moving task to target stage."""
    pass
```

**Output Schema**:
```yaml
move_proposal:
  task_id: "0042"
  from_stage: "current"
  to_stage: "integration"
  validation_result: "pass"
  proposed_events:
    - event: "implemented"
      desc: "OAuth2 flow implemented"
  checklist:
    - item: "Acceptance criteria met"
      checked: true
    - item: "Tests passing"
      checked: true
    - item: "Documentation updated"
      checked: true
  requires_human_approval: true
```

**Worker Responsibility**:
- Run validation checks
- Verify all stage exit criteria met
- **Propose** move, wait for human approval
- Do not modify task frontmatter directly

---

## Worker Protocol

### Standard Workflow

**Phase 1: Task Assignment**
1. Human assigns task to worker (or worker polls for available tasks)
2. Worker reads task package (all 3 files)
3. Worker declares capability match (can it handle this task type?)

**Phase 2: Planning**
4. Worker executes `plan()` capability
5. Worker proposes plan to human
6. Human reviews and approves/modifies plan

**Phase 3: Execution**
7. Worker executes `execute()` with approved plan
8. Worker logs progress events (not to ledger, to separate log file)
9. Worker proposes completion state

**Phase 4: Verification**
10. Human (or review worker) executes `review()`
11. If approved: Human writes `implemented` event to ledger
12. If rejected: Worker executes `execute()` again with feedback

**Phase 5: Handoff**
13. Worker executes `propose_move()` to next stage
14. Human approves move, updates frontmatter and index
15. Human writes stage transition event to ledger

### Idempotency Contract

**Workers MUST NOT**:
- Write to task ledgers directly
- Modify task frontmatter directly
- Update INDEX.md files directly
- Move tasks between stages directly

**Workers SHOULD**:
- Propose events (human writes to ledger after approval)
- Suggest frontmatter updates (human applies after review)
- Recommend stage transitions (human executes workflow)

**Why**: Preserve human control, enable rollback, maintain audit trail

---

## Error Contract

### Standard Error Types

**`validation_failed`**:
```yaml
error: "validation_failed"
task_id: "0042"
check: "dependencies_ready"
reason: "TASK-0010 is blocked"
remediation: "Wait for TASK-0010 to unblock"
```

**`dependency_unmet`**:
```yaml
error: "dependency_unmet"
task_id: "0042"
missing_dependency: "TASK-0010"
reason: "Required for API credentials"
```

**`test_failed`**:
```yaml
error: "test_failed"
task_id: "0042"
failing_tests:
  - "test_oauth2_flow"
  - "test_token_refresh"
error_message: "AssertionError: Expected 200, got 401"
```

**`timeout`**:
```yaml
error: "timeout"
task_id: "0042"
operation: "execute"
time_limit_minutes: 30
elapsed_minutes: 35
```

**`resource_limit_exceeded`**:
```yaml
error: "resource_limit_exceeded"
task_id: "0042"
limit_type: "max_files_per_task"
limit_value: 50
actual_value: 75
```

### Error Handling

**Worker Responsibility**:
- Detect errors early (fail fast)
- Provide clear error messages
- Suggest remediation steps
- Log errors in proposed events

**Human Responsibility**:
- Review error details
- Decide remediation (fix task, adjust plan, reassign)
- Log error event in ledger if needed

---

## Sandbox and Safety

### Worker Sandbox

**Purpose**: Isolate worker execution from production systems

**Sandbox Rules**:
- Workers execute in isolated environment (container, VM, etc.)
- Limited file system access (only task workspace)
- No network access to production systems
- Resource limits enforced (CPU, memory, time)

**Enforcement**:
- Runtime: Docker, VM, cloud sandbox
- File access: Whitelist (`/workspace`, `.work/tasks/{task_id}`)
- Network: Firewall rules, VPN isolation
- Resources: cgroups, ulimit

### Approval Gates

**Always Require Human Approval**:
- Moving task between stages
- Writing to production systems
- Deleting files
- Modifying dependencies
- Canceling or blocking tasks

**May Auto-Approve** (if configured):
- Reading files
- Running tests in sandbox
- Creating temporary files
- Proposing events (not writing)

---

## Worker Adapter Pattern

### Adapting Existing Agents

**Example: Claude/GPT Adapter**

```python
class ClaudeWorker:
    """Adapter for Claude/GPT to task management system."""
    
    def __init__(self, api_key: str):
        self.client = Anthropic(api_key=api_key)
        self.manifest = load_manifest("claude-3.5-sonnet")
    
    def plan(self, task_package: TaskPackage) -> Plan:
        """Generate execution plan."""
        prompt = f"""
        Task: {task_package.task.goal}
        Context: {task_package.context}
        
        Create a detailed execution plan with steps, time estimates, and risks.
        """
        response = self.client.messages.create(
            model="claude-3-5-sonnet-20241022",
            messages=[{"role": "user", "content": prompt}]
        )
        return parse_plan(response.content)
    
    def execute(self, task_package: TaskPackage, plan: Plan) -> ExecutionResult:
        """Execute task with tools."""
        # Use Claude's tool use capabilities
        tools = self._get_tools()  # read_file, write_file, etc.
        return self._execute_with_tools(task_package, plan, tools)
    
    def validate(self, task_package: TaskPackage, action: str) -> ValidationResult:
        """Run validation checks."""
        return run_validation_workflow(task_package, action)
```

**Example: Script/Automation Adapter**

```python
class ScriptWorker:
    """Adapter for automation scripts."""
    
    def execute(self, task_package: TaskPackage, plan: Plan) -> ExecutionResult:
        """Execute predefined script."""
        script = find_script_for_task(task_package.task.tags)
        result = subprocess.run([script, task_package.task_id], 
                                capture_output=True)
        return parse_execution_result(result)
```

---

## Worker Discovery and Registration

### Worker Registry

**Location**: `.work/tasks/.workers/`

**Registry Entry** (`{worker_id}.manifest.yaml`):
```yaml
worker_id: "claude-3.5-sonnet"
status: "active"
registered: "2025-10-18T10:00:00Z"
capabilities: [plan, validate, execute, review, summarize, propose_move]
task_types: ["implementation", "bugfix", "refactor"]
availability: "on_demand"  # or "scheduled", "always_on"
```

### Worker Assignment

**Manual Assignment**:
```yaml
# In task frontmatter
assigned_worker: "claude-3.5-sonnet"
```

**Automatic Assignment**:
```python
def assign_worker(task: Task) -> str:
    """Auto-assign worker based on task type."""
    if task.priority == "P0":
        return "human"  # P0 always human
    if task.tags.contains("implementation"):
        return "claude-3.5-sonnet"
    if task.tags.contains("review"):
        return "review-bot"
    return "human"
```

---

## Example: Full Worker Session

### Task: TASK-0042 - Implement OAuth2 Authentication

**Step 1: Assignment**
```yaml
# Human assigns task
assigned_worker: "claude-3.5-sonnet"
```

**Step 2: Worker Reads Package**
```python
task_pkg = load_task_package("0042")
# Reads: 0042-user-authentication.task.md
#        0042.context.md
#        0042.ledger.jsonl
```

**Step 3: Worker Plans**
```yaml
# Worker proposes plan
plan:
  steps: [...]
  estimated_effort: "M"
  risks: [...]

# Human reviews and approves
approved: true
```

**Step 4: Worker Executes**
```python
result = worker.execute(task_pkg, plan)
# Creates: src/auth/oauth2.py
#          tests/test_auth.py
# Proposes: implemented event
```

**Step 5: Human Reviews**
```yaml
# Human reviews artifacts
review: "approved"

# Human writes to ledger
{"ts":"2025-10-18T12:00:00Z","user":"alice","event":"implemented","desc":"OAuth2 flow by claude-3.5-sonnet"}
```

**Step 6: Worker Proposes Move**
```yaml
# Worker proposes move to integration
move_proposal:
  from_stage: "current"
  to_stage: "integration"
  validation: "pass"

# Human approves and executes WF5
```

---

## Implementation Notes

### For Human Execution (Manual Worker)

**Human as Worker**:
- Human reads task package
- Human creates plan (mental or written)
- Human implements task
- Human validates own work
- Human writes events to ledger manually

**No automation required** - system works without worker agents

### For Automation (Worker Agents)

**Worker Runtime**:
- Worker daemon polls for assigned tasks
- Worker executes capabilities as requested
- Worker proposes changes, waits for approval
- Approval queue managed by humans or approval bot

**Worker Orchestration**:
```python
class WorkerOrchestrator:
    def __init__(self):
        self.workers = load_worker_registry()
        self.approval_queue = []
    
    def assign_and_execute(self, task_id: str):
        worker = self.select_worker(task_id)
        task_pkg = load_task_package(task_id)
        
        # Planning phase
        plan = worker.plan(task_pkg)
        self.approval_queue.append(("plan", task_id, plan))
        # Wait for human approval...
        
        # Execution phase
        result = worker.execute(task_pkg, approved_plan)
        self.approval_queue.append(("execution", task_id, result))
        # Wait for human approval...
```

---

## References

- Query interface: `query.spec.md`
- Metrics schema: `metrics.schema.md`
- Event schema: `events.schema.md`
- Validation workflow: `workflows/validation.workflow.md`
- Workflow integration: `tasks.workflow.md`
- WIP policy: `wip.policy.md`
