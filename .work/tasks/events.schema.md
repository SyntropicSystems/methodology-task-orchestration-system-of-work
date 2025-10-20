---
type: schema
category: extensibility
last_updated: 2025-10-18
---

# Events Schema - Event Versioning & Hooks

This schema defines the event envelope format, versioning rules, and hook points for extending the task management system without breaking existing functionality.

## Principles

1. **Append-Only**: Events are immutable once written
2. **Versioned**: Each event has a version field for evolution
3. **Extensible**: New event types and fields can be added
4. **Backward Compatible**: Old events remain valid
5. **Hookable**: Key points allow custom logic injection

## Event Envelope Format

### Current Schema (v1.0)

**Minimal Format** (current):
```json
{"ts":"2025-10-18T10:30:00Z","user":"alice","event":"created","desc":"Task created"}
```

**Extended Format** (v1.0 with envelope):
```json
{
  "version": "1.0",
  "ts": "2025-10-18T10:30:00Z",
  "user": "alice",
  "event": "created",
  "desc": "Task created",
  "metadata": {
    "source": "cli",
    "worker_id": null
  }
}
```

### Required Fields (All Versions)

- **ts**: ISO 8601 timestamp with timezone (e.g., `2025-10-18T10:30:00Z`)
- **user**: User identifier (e.g., `alice`, `@alice`, `user_123`)
- **event**: Event type from canonical vocabulary
- **desc**: Human-readable description of what happened

### Optional Fields (v1.0+)

- **version**: Schema version (default: `1.0` if omitted)
- **metadata**: Additional context (source, worker_id, etc.)
- **payload**: Event-specific structured data

## Versioning Rules

### Version Number Format

`MAJOR.MINOR` (e.g., `1.0`, `1.1`, `2.0`)

### Version Increment Rules

**MINOR Version** (backward compatible):
- Add new optional fields
- Add new event types
- Extend metadata schema
- Add new hook points

**MAJOR Version** (breaking changes):
- Remove required fields
- Change field semantics
- Rename fields
- Change event type meanings

### Compatibility Matrix

| Writer Version | Reader Version | Compatible? | Notes |
|----------------|----------------|-------------|-------|
| 1.0 | 1.0 | ✓ | Perfect match |
| 1.1 | 1.0 | ✓ | Reader ignores new fields |
| 1.0 | 1.1 | ✓ | New fields get defaults |
| 2.0 | 1.0 | ✗ | Breaking change |
| 1.0 | 2.0 | ✗ | Breaking change |

### Version Detection

**Reading Events**:
```python
def read_event(line: str) -> Event:
    """Parse event from JSONL line."""
    data = json.loads(line)
    version = data.get("version", "1.0")  # Default to 1.0 if missing
    
    if version == "1.0":
        return parse_v1_event(data)
    elif version == "2.0":
        return parse_v2_event(data)
    else:
        raise UnsupportedVersionError(f"Unknown event version: {version}")
```

**Writing Events**:
```python
def write_event(ledger_path: str, event: Event):
    """Write event to ledger with version."""
    data = {
        "version": "1.0",
        "ts": event.timestamp,
        "user": event.user,
        "event": event.event_type,
        "desc": event.description
    }
    with open(ledger_path, "a") as f:
        f.write(json.dumps(data) + "\n")
```

## Event Vocabulary Evolution

### Current Vocabulary (v1.0)

**Core Lifecycle**:
- `id_reserved`, `created`, `verified`, `prioritized`, `started`, `implemented`, `integrated`, `learning_documented`, `learning_skipped`, `closed`

**State Management**:
- `blocked`, `unblocked`, `paused`, `resumed`, `canceled`

**Hand-off**:
- `handoff_initiated`, `handoff_accepted`

**Graph Operations**:
- `split`, `merged_from`, `dependency_updated`

**Failure Handling**:
- `rollback`, `rework_required`

**Content & Metadata**:
- `content_updated`, `priority_set`, `priority_changed`, `effort_estimated`, `tags_updated`

**System**:
- `index_synced`, `index_drift_detected`, `move_blocked`

### Adding New Event Types (Minor Version)

**Example: Add `ai_reviewed` event in v1.1**:

```json
{"version":"1.1","ts":"2025-10-18T10:30:00Z","user":"claude-3.5-sonnet","event":"ai_reviewed","desc":"AI review completed with suggestions"}
```

**Backward Compatibility**:
- v1.0 readers see it as generic event
- v1.1 readers can parse and handle specifically
- No breaking changes to existing events

**Documentation**:
```markdown
### v1.1 (2025-10-20)

**New Events**:
- `ai_reviewed`: AI worker completed review with feedback
- `ai_plan_proposed`: AI worker proposed execution plan

**New Fields**:
- `metadata.worker_id`: Identifier of worker that generated event
- `metadata.confidence`: AI confidence score (0.0-1.0)
```

## Hook Points

Hooks allow custom logic to run at key points in the system without modifying core workflows.

### Hook Definition

**Hook Contract**:
```python
def hook_function(event: Event, task_pkg: TaskPackage) -> List[Event]:
    """
    Process hook invocation.
    
    Args:
        event: The triggering event
        task_pkg: Current task package state
    
    Returns:
        List of proposed events to append (may be empty)
    """
    pass
```

### Core Hook Points

#### 1. `on_validate_pass`
**Trigger**: After WF-VALIDATE-MOVE passes  
**Use Case**: Additional validation, logging, notifications

**Example**:
```python
def on_validate_pass(event, task_pkg):
    """Notify team when validation passes for high-priority tasks."""
    if task_pkg.task.priority == "P0":
        send_notification(f"P0 task {task_pkg.task_id} validated for move")
    return []  # No additional events
```

#### 2. `on_move`
**Trigger**: After task moves between stages  
**Use Case**: Metrics collection, notifications, automation triggers

**Example**:
```python
def on_move(event, task_pkg):
    """Update WIP metrics when task moves."""
    old_stage = extract_stage_from_event(event)
    new_stage = task_pkg.task.stage
    update_wip_metrics(old_stage, new_stage)
    return []
```

#### 3. `on_blocked`
**Trigger**: When task becomes blocked  
**Use Case**: Blocker tracking, escalation, notifications

**Example**:
```python
def on_blocked(event, task_pkg):
    """Escalate if P1 task blocked for >24 hours."""
    if task_pkg.task.priority == "P1":
        blocked_duration = compute_blocked_duration(task_pkg)
        if blocked_duration > 24 * 3600:  # 24 hours
            return [{
                "event": "escalated",
                "desc": f"P1 blocked for {blocked_duration/3600:.1f} hours"
            }]
    return []
```

#### 4. `on_slo_missed`
**Trigger**: When SLO target exceeded  
**Use Case**: Alerting, root cause analysis, capacity planning

**Example**:
```python
def on_slo_missed(event, task_pkg):
    """Alert team and log SLO miss for analysis."""
    send_alert(f"SLO missed: {task_pkg.task_id} - {event.desc}")
    log_slo_miss_for_analysis(task_pkg, event)
    return []
```

#### 5. `on_p0_created`
**Trigger**: When P0 task is created  
**Use Case**: Immediate notification, on-call paging, displacement

**Example**:
```python
def on_p0_created(event, task_pkg):
    """Page on-call engineer for P0."""
    page_oncall(f"P0 created: {task_pkg.task_id}")
    check_displacement_needed()
    return []
```

#### 6. `on_completion`
**Trigger**: When task closes (archived)  
**Use Case**: Metrics update, celebration, learning extraction

**Example**:
```python
def on_completion(event, task_pkg):
    """Update throughput metrics and celebrate."""
    update_throughput_metrics(task_pkg)
    if task_pkg.task.priority == "P0":
        return [{
            "event": "p0_postmortem_required",
            "desc": "P0 completion requires postmortem"
        }]
    return []
```

#### 7. `on_rollback`
**Trigger**: When task is rolled back to previous stage  
**Use Case**: Rework tracking, quality analysis, learning

**Example**:
```python
def on_rollback(event, task_pkg):
    """Track rework rate and analyze patterns."""
    update_rework_metrics(task_pkg)
    analyze_rollback_reason(event.desc)
    return []
```

#### 8. `on_dependency_updated`
**Trigger**: When task dependencies change  
**Use Case**: Cycle detection, blocker analysis, impact tracking

**Example**:
```python
def on_dependency_updated(event, task_pkg):
    """Detect cycles when dependencies change."""
    if detect_cycle(task_pkg.task_id):
        return [{
            "event": "cycle_detected",
            "desc": "Circular dependency introduced"
        }]
    return []
```

### Hook Registration

**Hook Registry** (`.work/tasks/.hooks/registry.yaml`):
```yaml
hooks:
  on_validate_pass:
    - name: "p0_notification"
      function: "hooks.notifications.on_validate_pass"
      enabled: true
  on_move:
    - name: "wip_metrics"
      function: "hooks.metrics.on_move"
      enabled: true
  on_blocked:
    - name: "blocker_escalation"
      function: "hooks.escalation.on_blocked"
      enabled: true
  on_slo_missed:
    - name: "slo_alerting"
      function: "hooks.alerts.on_slo_missed"
      enabled: true
  on_p0_created:
    - name: "oncall_paging"
      function: "hooks.oncall.on_p0_created"
      enabled: true
  on_completion:
    - name: "completion_metrics"
      function: "hooks.metrics.on_completion"
      enabled: true
  on_rollback:
    - name: "rework_tracking"
      function: "hooks.quality.on_rollback"
      enabled: true
  on_dependency_updated:
    - name: "cycle_detection"
      function: "hooks.graph.on_dependency_updated"
      enabled: true
```

### Hook Execution

**Hook Runner**:
```python
class HookRunner:
    def __init__(self):
        self.registry = load_hook_registry()
    
    def run_hooks(self, hook_point: str, event: Event, task_pkg: TaskPackage) -> List[Event]:
        """Execute all hooks for a hook point."""
        proposed_events = []
        
        hooks = self.registry.get(hook_point, [])
        for hook_config in hooks:
            if not hook_config["enabled"]:
                continue
            
            hook_fn = load_function(hook_config["function"])
            try:
                new_events = hook_fn(event, task_pkg)
                proposed_events.extend(new_events)
            except Exception as e:
                log_error(f"Hook {hook_config['name']} failed: {e}")
        
        return proposed_events
```

**Hook Invocation** (in workflows):
```python
def execute_wf4_move_to_current(task_id):
    """WF4: Move task to current stage."""
    task_pkg = load_task_package(task_id)
    
    # 1. Run validation
    validation = validate_move(task_pkg, "current")
    if not validation.passed:
        return validation.error
    
    # 2. Run on_validate_pass hooks
    hook_events = run_hooks("on_validate_pass", validation.event, task_pkg)
    
    # 3. Move task
    move_task(task_id, "current")
    
    # 4. Run on_move hooks
    move_event = create_event("started", "Task moved to current")
    hook_events.extend(run_hooks("on_move", move_event, task_pkg))
    
    # 5. Write events (proposed by hooks) after human approval
    return {"success": True, "proposed_events": hook_events}
```

## Event Payload Schema

For complex events, use structured payload field.

### Payload Examples

**WF-SPLIT Event**:
```json
{
  "version": "1.0",
  "ts": "2025-10-18T10:30:00Z",
  "user": "alice",
  "event": "split",
  "desc": "Split into 3 subtasks",
  "payload": {
    "parent_id": "0042",
    "child_ids": ["0043", "0044", "0045"],
    "split_reason": "Scope too large",
    "dependency_inheritance": "default"
  }
}
```

**WF-VALIDATE-MOVE Event**:
```json
{
  "version": "1.0",
  "ts": "2025-10-18T10:30:00Z",
  "user": "system",
  "event": "move_blocked",
  "desc": "Move validation failed",
  "payload": {
    "task_id": "0042",
    "from_stage": "next",
    "to_stage": "current",
    "validation_result": "fail",
    "failed_checks": [
      {
        "check": "dependencies_ready",
        "reason": "TASK-0010 is blocked"
      }
    ]
  }
}
```

**AI Review Event** (v1.1):
```json
{
  "version": "1.1",
  "ts": "2025-10-18T10:30:00Z",
  "user": "claude-3.5-sonnet",
  "event": "ai_reviewed",
  "desc": "AI review completed",
  "metadata": {
    "worker_id": "claude-3.5-sonnet",
    "confidence": 0.92
  },
  "payload": {
    "result": "approve_with_suggestions",
    "suggestions": [
      "Add rate limiting",
      "Consider token rotation"
    ],
    "acceptance_criteria_met": true
  }
}
```

## Event Schema Migration

### Migrating from v1.0 to v1.1

**Scenario**: Add `metadata.worker_id` field in v1.1

**Migration Strategy**:
1. v1.1 writers include `metadata.worker_id`
2. v1.0 readers ignore unknown fields (forward compatible)
3. v1.1 readers handle missing field with default (backward compatible)

**No migration script needed** - both versions coexist

### Migrating from v1.x to v2.0 (Breaking)

**Scenario**: Rename `user` field to `actor` in v2.0

**Migration Strategy**:
1. Deprecate v1.x, announce v2.0
2. Provide migration script to transform ledgers
3. Support dual-read for transition period

**Migration Script**:
```python
def migrate_ledger_v1_to_v2(ledger_path: str):
    """Migrate ledger from v1.x to v2.0."""
    events = []
    with open(ledger_path, "r") as f:
        for line in f:
            event = json.loads(line)
            version = event.get("version", "1.0")
            
            if version.startswith("1."):
                # Transform v1 -> v2
                event["version"] = "2.0"
                event["actor"] = event.pop("user")  # Rename field
            
            events.append(event)
    
    # Write back
    with open(ledger_path, "w") as f:
        for event in events:
            f.write(json.dumps(event) + "\n")
```

## Hook Development Guide

### Creating a Custom Hook

**1. Define Hook Function**:
```python
# hooks/custom/slack_notifier.py

def on_p0_created(event: Event, task_pkg: TaskPackage) -> List[Event]:
    """Send Slack notification when P0 is created."""
    message = f"🚨 P0 Alert: {task_pkg.task_id} - {task_pkg.task.task_name}"
    send_slack_message("#oncall", message)
    
    # Optionally propose additional events
    return [{
        "event": "slack_notification_sent",
        "desc": f"Notified #oncall about P0 {task_pkg.task_id}"
    }]
```

**2. Register Hook**:
```yaml
# .work/tasks/.hooks/registry.yaml
hooks:
  on_p0_created:
    - name: "slack_p0_notifier"
      function: "hooks.custom.slack_notifier.on_p0_created"
      enabled: true
```

**3. Test Hook**:
```python
def test_slack_notifier():
    """Test P0 Slack notification hook."""
    event = create_test_event("created")
    task_pkg = create_test_task_package(priority="P0")
    
    proposed_events = on_p0_created(event, task_pkg)
    
    assert len(proposed_events) == 1
    assert proposed_events[0]["event"] == "slack_notification_sent"
```

### Hook Best Practices

1. **Keep Hooks Fast**: Hooks run synchronously, don't block workflows
2. **Handle Errors Gracefully**: Hook failures shouldn't break workflows
3. **Propose Events, Don't Write**: Return proposed events for approval
4. **Be Idempotent**: Hooks may be called multiple times
5. **Log Hook Activity**: Log all hook executions for debugging

## Event Query and Filtering

### Query Events by Type

```python
def get_events_by_type(ledger_path: str, event_type: str) -> List[Event]:
    """Get all events of specific type from ledger."""
    events = []
    with open(ledger_path, "r") as f:
        for line in f:
            event = json.loads(line)
            if event["event"] == event_type:
                events.append(event)
    return events
```

### Query Events by Time Range

```python
def get_events_in_range(ledger_path: str, start: str, end: str) -> List[Event]:
    """Get all events within time range."""
    events = []
    with open(ledger_path, "r") as f:
        for line in f:
            event = json.loads(line)
            if start <= event["ts"] <= end:
                events.append(event)
    return events
```

### Event Stream Processing

```python
def process_event_stream(ledger_path: str, processor: Callable):
    """Process events as stream."""
    with open(ledger_path, "r") as f:
        for line in f:
            event = json.loads(line)
            processor(event)
```

## Implementation Notes

### For Human Execution

**Minimal Hook Support**:
- No hooks initially
- Add hooks as automation evolves
- Manual invocation (e.g., "check if P0, send notification")

**Tools**: No tooling required for core events

### For Automation

**Hook System**:
- Hook registry with enable/disable per hook
- Hook runner integrated into workflow execution
- Asynchronous hooks for slow operations (notifications, API calls)
- Hook error handling and retries

**Event Versioning**:
- Version field in all new events
- Version detection in readers
- Migration scripts for breaking changes

## References

- Query interface: `query.spec.md`
- Worker contract: `worker.contract.md`
- Metrics schema: `metrics.schema.md`
- WIP policy: `wip.policy.md`
- Workflow integration: `tasks.workflow.md`
