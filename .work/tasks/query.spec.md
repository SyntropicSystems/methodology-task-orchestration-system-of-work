---
type: specification
category: data_access
last_updated: 2025-10-18
---

# Query Specification - Query Layer

This specification defines the query interface for accessing task management system data. The query layer provides consistent, structured access to task data for humans, agents, and automation.

## Principles

1. **Single Source of Truth**: All queries read from canonical data sources (frontmatter, indexes, ledgers)
2. **Declarative**: Queries specify what data is needed, not how to retrieve it
3. **Consistent Interface**: Same query works for CLI, API, file-based access
4. **Read-Only**: Queries never modify state (use workflows for mutations)
5. **Composable**: Queries can be combined to answer complex questions

## Data Sources

### Primary Sources

**Task Frontmatter** (`{id}-{name}.task.md` YAML):
- Current task state (stage, status, priority, effort, DRI)
- Dependencies (depends_on, blocks, related_to)
- Split relationships (split_from, split_into)
- Metadata (tags, dates, schema version)

**Index Files** (`INDEX.md`):
- Stage membership (which tasks in each stage)
- Quick status overview (active, blocked, paused, canceled)
- Dependency references

**Event Ledgers** (`{id}.ledger.jsonl`):
- Complete event history (what happened, when, by whom)
- State transitions (stage movements, status changes)
- Duration calculations (time between events)

### Derived Sources

**Metrics** (computed from ledgers):
- Lead times, flow times, blocked times
- Throughput, WIP, arrival rates
- See `metrics.schema.md` for definitions

## Canonical Queries

### 1. Get Task by ID

**Purpose**: Retrieve complete task details  
**Input**: Task ID (e.g., "0042")  
**Output**: Task object with all fields

**Query**:
```
get_task(task_id: str) -> Task
```

**Example Response**:
```json
{
  "task_id": "0042",
  "task_name": "User Authentication",
  "stage": "current",
  "status": "blocked",
  "priority": "P1",
  "effort_estimate": "M",
  "current_dri": "@alice",
  "created_by": "@bob",
  "created_date": "2025-10-01",
  "depends_on": ["TASK-0010", "TASK-0023"],
  "blocks": ["TASK-0055"],
  "blocked_by": "TASK-0010",
  "tags": ["authentication", "security"],
  "schema_version": "1.0"
}
```

**Data Source**: Parse `0042-user-authentication.task.md` frontmatter

---

### 2. Get Tasks by Stage

**Purpose**: List all tasks in a specific stage  
**Input**: Stage name (e.g., "current")  
**Output**: List of task IDs and summary info

**Query**:
```
get_tasks_by_stage(stage: str) -> List[TaskSummary]
```

**Example Response**:
```json
[
  {
    "task_id": "0042",
    "task_name": "User Authentication",
    "dri": "@alice",
    "status": "blocked",
    "priority": "P1"
  },
  {
    "task_id": "0043",
    "task_name": "Payment Gateway",
    "dri": "@bob",
    "status": "active",
    "priority": "P2"
  }
]
```

**Data Source**: Parse `{stage}/INDEX.md` table

---

### 3. Get Blocked Tasks

**Purpose**: Find all tasks currently blocked  
**Input**: None (or optional: filter by stage)  
**Output**: List of blocked tasks with blocker info

**Query**:
```
get_blocked_tasks(stage: Optional[str] = None) -> List[BlockedTask]
```

**Example Response**:
```json
[
  {
    "task_id": "0042",
    "task_name": "User Authentication",
    "stage": "current",
    "blocked_by": "TASK-0010",
    "blocked_since": "2025-10-15T10:30:00Z",
    "blocked_duration_hours": 72,
    "dri": "@alice"
  },
  {
    "task_id": "0049",
    "task_name": "Analytics Dashboard",
    "stage": "current",
    "blocked_by": "external:vendor_approval",
    "blocked_since": "2025-10-13T09:00:00Z",
    "blocked_duration_hours": 120,
    "dri": "@charlie"
  }
]
```

**Data Source**: 
1. Query tasks with `status: blocked` from frontmatter
2. Find most recent `blocked` event in ledger for `blocked_since`
3. Compute duration: NOW() - blocked_since

---

### 4. Get Task Dependents

**Purpose**: Find all tasks that depend on a given task (downstream)  
**Input**: Task ID  
**Output**: List of dependent task IDs

**Query**:
```
get_dependents(task_id: str) -> List[str]
```

**Example**:
```
get_dependents("0010") -> ["0042", "0049", "0055"]
```

**Data Source**: 
1. Query all task frontmatter files
2. Find tasks with task_id in their `depends_on` array
3. Return list of dependent task IDs

**Inverse Query**: `get_dependencies(task_id)` returns tasks in the `depends_on` field

---

### 5. Get Task Dependencies (Upstream)

**Purpose**: Find all tasks that a given task depends on (upstream)  
**Input**: Task ID  
**Output**: List of dependency task IDs

**Query**:
```
get_dependencies(task_id: str) -> List[str]
```

**Example**:
```
get_dependencies("0042") -> ["0010", "0023"]
```

**Data Source**: Read `depends_on` field from task frontmatter

---

### 6. Get Stale Tasks

**Purpose**: Find tasks exceeding SLO targets  
**Input**: Optional stage filter  
**Output**: List of stale tasks with SLO miss info

**Query**:
```
get_stale_tasks(stage: Optional[str] = None) -> List[StaleTask]
```

**Example Response**:
```json
[
  {
    "task_id": "0042",
    "task_name": "User Authentication",
    "stage": "current",
    "time_in_stage_hours": 240,
    "slo_target_hours": 120,
    "slo_miss_ratio": 2.0,
    "dri": "@alice"
  }
]
```

**Data Source**:
1. Parse task frontmatter for current stage
2. Find stage entry event in ledger (e.g., `started` for current)
3. Compute time_in_stage: NOW() - entry_timestamp
4. Compare to SLO target from `wip.policy.md`

---

### 7. Get WIP by Stage

**Purpose**: Count active tasks in each stage  
**Input**: None  
**Output**: WIP counts per stage

**Query**:
```
get_wip_by_stage() -> Dict[str, int]
```

**Example Response**:
```json
{
  "inbox": 2,
  "backlog": 8,
  "next": 6,
  "current": 4,
  "integration": 2,
  "learning": 1,
  "archived": 0
}
```

**Data Source**: Count rows in each stage's `INDEX.md` where status = "active"

---

### 8. Get P0 Status

**Purpose**: Find all active P0 tasks and their status  
**Input**: None  
**Output**: List of P0 tasks

**Query**:
```
get_p0_tasks() -> List[TaskSummary]
```

**Example Response**:
```json
[
  {
    "task_id": "0055",
    "task_name": "API Outage Recovery",
    "stage": "current",
    "status": "active",
    "created_date": "2025-10-18",
    "dri": "@alice",
    "time_since_creation_hours": 2
  }
]
```

**Data Source**: Query all tasks with `priority: P0` and `status: active`

---

### 9. Get Rework History

**Purpose**: Find tasks with rollback events  
**Input**: Optional time range  
**Output**: List of tasks with rework details

**Query**:
```
get_rework_history(start_date: Optional[str] = None, end_date: Optional[str] = None) -> List[ReworkTask]
```

**Example Response**:
```json
[
  {
    "task_id": "0048",
    "task_name": "Payment Validation",
    "rollback_count": 1,
    "rollback_events": [
      {
        "timestamp": "2025-10-17T14:30:00Z",
        "from_stage": "integration",
        "to_stage": "current",
        "reason": "Acceptance criteria changed",
        "user": "@bob"
      }
    ]
  }
]
```

**Data Source**: Parse ledgers for `rollback` events in date range

---

### 10. Get Task Events

**Purpose**: Retrieve complete event history for a task  
**Input**: Task ID, optional event type filter  
**Output**: List of events

**Query**:
```
get_task_events(task_id: str, event_type: Optional[str] = None) -> List[Event]
```

**Example Response**:
```json
[
  {
    "ts": "2025-10-01T10:00:00Z",
    "user": "alice",
    "event": "created",
    "desc": "New feature request"
  },
  {
    "ts": "2025-10-01T14:30:00Z",
    "user": "bob",
    "event": "verified",
    "desc": "Package complete"
  }
]
```

**Data Source**: Parse `{task_id}.ledger.jsonl`

---

### 11. Get Blocker Impact

**Purpose**: Find tasks causing the most downstream blocking  
**Input**: Optional limit (default: top 10)  
**Output**: Tasks ranked by blocker impact

**Query**:
```
get_blocker_impact(limit: int = 10) -> List[BlockerImpact]
```

**Example Response**:
```json
[
  {
    "blocker_id": "0010",
    "blocker_name": "API Access Setup",
    "blocking_count": 3,
    "total_blocked_hours": 216,
    "blocked_tasks": ["0042", "0049", "0055"]
  },
  {
    "blocker_id": "0023",
    "blocker_name": "Database Schema Update",
    "blocking_count": 1,
    "total_blocked_hours": 48,
    "blocked_tasks": ["0049"]
  }
]
```

**Data Source**:
1. Find all blocked tasks via `get_blocked_tasks()`
2. Group by `blocked_by` field
3. Sum blocked durations per blocker
4. Sort by total_blocked_hours descending

---

### 12. Compute Metrics

**Purpose**: Calculate metrics for tasks or time periods  
**Input**: Metric name, optional filters  
**Output**: Metric value(s)

**Queries**:
```
compute_lead_time(task_id: str) -> float
compute_flow_time(task_id: str, stage: str) -> float
compute_throughput(start_date: str, end_date: str) -> int
compute_wip(stage: str, point_in_time: str) -> int
```

**Example**:
```
compute_lead_time("0042") -> 8.19  # days
compute_throughput("2025-10-14", "2025-10-20") -> 15  # tasks
```

**Data Source**: Parse ledgers, apply formulas from `metrics.schema.md`

---

## Query Output Contracts

### Tabular Format (CSV/Markdown)

For human consumption, CLI output:

```
| ID   | Name                | Stage    | Status  | DRI    |
|------|---------------------|----------|---------|--------|
| 0042 | User Authentication | current  | blocked | @alice |
| 0043 | Payment Gateway     | current  | active  | @bob   |
```

### JSON Format (API/Agent)

For programmatic consumption:

```json
[
  {
    "task_id": "0042",
    "task_name": "User Authentication",
    "stage": "current",
    "status": "blocked",
    "dri": "@alice"
  }
]
```

### Event Stream Format (Real-time)

For live monitoring, webhooks:

```json
{
  "query": "get_blocked_tasks",
  "timestamp": "2025-10-18T10:30:00Z",
  "results": [...]
}
```

## Query Interface Options

### Option 1: CLI Tool (Human-Friendly)

```bash
# Get blocked tasks
task-cli query blocked

# Get task by ID
task-cli query task 0042

# Get WIP by stage
task-cli query wip

# Get stale tasks
task-cli query stale --stage current
```

### Option 2: File-Based Query (Agent-Friendly)

**Query File**: `.work/tasks/.query.yaml`
```yaml
query: get_blocked_tasks
filters:
  stage: current
```

**Result File**: `.work/tasks/.query_result.json`
```json
{
  "timestamp": "2025-10-18T10:30:00Z",
  "query": "get_blocked_tasks",
  "results": [...]
}
```

**Workflow**:
1. Agent writes query specification to `.query.yaml`
2. System processes query (human or automation)
3. System writes results to `.query_result.json`
4. Agent reads results

### Option 3: API (Automation)

```http
GET /api/tasks?stage=current&status=blocked
GET /api/tasks/0042
GET /api/wip
GET /api/metrics/throughput?start=2025-10-14&end=2025-10-20
```

### Option 4: SQL Interface (Advanced)

For complex queries, provide SQL interface over virtual schema:

```sql
-- Find tasks blocked for >3 days
SELECT task_id, task_name, blocked_by, 
       (NOW() - blocked_since) AS blocked_duration
FROM tasks
WHERE status = 'blocked'
  AND (NOW() - blocked_since) > INTERVAL '3 days'
ORDER BY blocked_duration DESC;

-- Throughput by priority
SELECT priority, COUNT(*) AS completed
FROM tasks
WHERE stage = 'archived'
  AND closed_date BETWEEN '2025-10-14' AND '2025-10-20'
GROUP BY priority;
```

## Query Composition

**Chaining Queries**:
```python
# Get blocked tasks in current stage, find their blockers
blocked = get_blocked_tasks(stage="current")
for task in blocked:
    blocker = get_task(task.blocked_by)
    print(f"{task.task_id} blocked by {blocker.task_id} ({blocker.stage})")
```

**Complex Query Example** (find critical path):
```python
def find_critical_path(task_id):
    """Find longest dependency chain to given task."""
    task = get_task(task_id)
    if not task.depends_on:
        return [task_id]
    
    max_path = []
    for dep_id in task.depends_on:
        path = find_critical_path(dep_id)
        if len(path) > len(max_path):
            max_path = path
    
    return max_path + [task_id]
```

## Query Performance

### For Human Execution
- Simple queries (get task, list stage): <1 second (read 1 file)
- Complex queries (blocker impact, metrics): <30 seconds (parse all ledgers)
- Use caching for repeated queries within same session

### For Automation
- Index builds: Pre-compute common queries (WIP, blocked tasks)
- Incremental updates: Update indexes on state changes
- Query cache: TTL-based cache for expensive queries (1-5 minutes)

**Example Index File** (`.work/tasks/.index_cache/blocked_tasks.json`):
```json
{
  "updated": "2025-10-18T10:30:00Z",
  "ttl_minutes": 5,
  "results": [...]
}
```

## Error Handling

**Standard Error Types**:
```json
{
  "error": "task_not_found",
  "task_id": "9999",
  "message": "Task TASK-9999 does not exist"
}

{
  "error": "invalid_stage",
  "stage": "unknown",
  "message": "Stage 'unknown' is not valid. Valid stages: inbox, backlog, next, current, integration, learning, archived"
}

{
  "error": "query_timeout",
  "query": "get_blocker_impact",
  "message": "Query exceeded 30 second timeout"
}
```

## Implementation Notes

### For Human Execution (Minimal Query Layer)

**Simple Queries**:
- List tasks in stage: Read `{stage}/INDEX.md` manually
- Get blocked tasks: Search for `status: blocked` in frontmatter
- Compute metrics: Use spreadsheet with formulas from `metrics.schema.md`

**Tools**: `grep`, `awk`, spreadsheet, or simple Python script

### For Automation (Full Query Layer)

**Query Engine**:
```python
class TaskQuery:
    def __init__(self, tasks_root=".work/tasks"):
        self.tasks_root = tasks_root
        self.cache = {}
    
    def get_task(self, task_id: str) -> Task:
        """Load task from frontmatter."""
        path = find_task_file(task_id)
        return parse_frontmatter(path)
    
    def get_blocked_tasks(self, stage=None) -> List[BlockedTask]:
        """Find all blocked tasks."""
        tasks = self._load_all_tasks()
        blocked = [t for t in tasks if t.status == "blocked"]
        if stage:
            blocked = [t for t in blocked if t.stage == stage]
        return blocked
    
    def _load_all_tasks(self):
        """Load all tasks (with caching)."""
        if "all_tasks" in self.cache:
            return self.cache["all_tasks"]
        # Scan all stage folders, parse frontmatter
        ...
```

**Query Optimization**:
- Build index on startup (scan all tasks once)
- Watch file system for changes, update index incrementally
- Use cache for repeated queries

## References

- Metrics definitions: `metrics.schema.md`
- WIP policy and SLOs: `wip.policy.md`
- Standard reports: `reports.spec.md`
- Worker contract: `worker.contract.md`
- Event schema: `events.schema.md`
- Workflow integration: `tasks.workflow.md`
