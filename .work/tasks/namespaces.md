---
type: specification
category: scalability
last_updated: 2025-10-18
---

# Namespaces - Multi-Team Scaling

This specification defines namespace support for scaling the task management system across multiple teams, products, or organizational units.

## Principles

1. **Isolation**: Each namespace has independent task numbering and workflow
2. **Cross-Reference**: Tasks can reference tasks in other namespaces
3. **Unified View**: System-wide queries can aggregate across namespaces
4. **Gradual Adoption**: Start with single namespace, add more as needed
5. **Namespace-Aware**: All queries and reports namespace-scoped by default

## Namespace ID Scheme

### Format

`{NAMESPACE}-{ID}` where:
- **NAMESPACE**: 2-6 character prefix (uppercase letters/numbers)
- **ID**: 4-digit zero-padded number (0001-9999)

### Examples

```
FW-0001      # Fireweed platform team, task 1
FW-0042      # Fireweed platform team, task 42
PLAT-0001    # Platform infrastructure team, task 1
PLAT-0023    # Platform infrastructure team, task 23
MOBILE-0015  # Mobile app team, task 15
API-0100     # API team, task 100
```

### Namespace Registry

**Location**: `.work/tasks/NAMESPACES.yaml`

**Format**:
```yaml
namespaces:
  FW:
    full_name: "Fireweed Platform"
    description: "Main product features and enhancements"
    team: "Product Engineering"
    created: "2025-10-01"
    id_counter_file: ".work/tasks/fw/ID_COUNTER.txt"
    
  PLAT:
    full_name: "Platform Infrastructure"
    description: "Infrastructure, DevOps, and platform services"
    team: "Platform Engineering"
    created: "2025-10-15"
    id_counter_file: ".work/tasks/plat/ID_COUNTER.txt"
    
  MOBILE:
    full_name: "Mobile Applications"
    description: "iOS and Android mobile apps"
    team: "Mobile Engineering"
    created: "2025-10-20"
    id_counter_file: ".work/tasks/mobile/ID_COUNTER.txt"
```

## Folder Structure Options

### Option 1: Unified Structure (Single Task Root)

**Recommended for small-to-medium organizations**

```
.work/tasks/
├── NAMESPACES.yaml
├── INDEX.md                    # System-wide index (all namespaces)
├── inbox/
│   ├── INDEX.md                # All namespaces in inbox
│   ├── TASK-FW-0001-feature/
│   ├── TASK-PLAT-0005-infra/
│   └── TASK-MOBILE-0012-bug/
├── backlog/
│   ├── INDEX.md
│   ├── TASK-FW-0002-enhancement/
│   └── TASK-PLAT-0006-migration/
├── next/
│   └── INDEX.md
├── current/
│   ├── INDEX.md
│   ├── TASK-FW-0003-api/
│   └── TASK-MOBILE-0013-ui/
├── integration/
│   └── INDEX.md
├── learning/
│   └── INDEX.md
└── archived/
    └── INDEX.md
```

**Advantages**:
- Simple structure
- Unified workflow across teams
- Easy cross-namespace queries
- Single set of governance files

**Disadvantages**:
- Can get crowded with many teams
- Harder to set team-specific permissions

---

### Option 2: Per-Namespace Structure

**Recommended for large organizations or strict team isolation**

```
.work/tasks/
├── NAMESPACES.yaml
├── INDEX.md                    # System-wide index
│
├── fw/                         # Fireweed namespace
│   ├── ID_COUNTER.txt
│   ├── README.md
│   ├── INDEX.md
│   ├── inbox/
│   │   ├── INDEX.md
│   │   └── TASK-FW-0001-feature/
│   ├── backlog/
│   ├── next/
│   ├── current/
│   ├── integration/
│   ├── learning/
│   └── archived/
│
├── plat/                       # Platform namespace
│   ├── ID_COUNTER.txt
│   ├── README.md
│   ├── INDEX.md
│   ├── inbox/
│   │   └── TASK-PLAT-0005-infra/
│   ├── backlog/
│   └── ...
│
└── mobile/                     # Mobile namespace
    ├── ID_COUNTER.txt
    ├── README.md
    ├── INDEX.md
    ├── inbox/
    └── ...
```

**Advantages**:
- Clear team boundaries
- Easier to set permissions per namespace
- Independent workflows per team
- Simpler to split repositories later

**Disadvantages**:
- More complex structure
- Cross-namespace queries require scanning multiple paths
- Duplicate governance files (or symlinks)

---

### Hybrid Option: Namespace Folders with Unified Stages

```
.work/tasks/
├── NAMESPACES.yaml
├── INDEX.md
├── inbox/
│   ├── fw/
│   │   └── TASK-FW-0001-feature/
│   ├── plat/
│   │   └── TASK-PLAT-0005-infra/
│   └── mobile/
│       └── TASK-MOBILE-0012-bug/
├── backlog/
│   ├── fw/
│   ├── plat/
│   └── mobile/
└── ...
```

**Advantages**: Balance of organization and simplicity

---

## Cross-Namespace Dependencies

### Fully-Qualified Task IDs

**Within same namespace**:
```yaml
depends_on:
  - "TASK-FW-0010"      # Same namespace (FW)
  - "TASK-FW-0015"
```

**Across namespaces**:
```yaml
depends_on:
  - "TASK-PLAT-0023"    # Platform namespace dependency
  - "TASK-FW-0010"      # Same namespace
```

### Dependency Validation

**Rule**: All dependency task IDs must exist and be accessible

**Cross-Namespace Access**:
- Tasks can depend on tasks in any namespace
- Both namespaces must be accessible to user
- Blocked/paused tasks in other namespaces block dependents

**Example**:
```
FW-0042 (Fireweed) depends on PLAT-0023 (Platform)
  ↓
If PLAT-0023 is blocked, FW-0042 is also blocked
```

### Cross-Namespace Blocker Impact

**Blocker Query** (see `query.spec.md`):
```python
def get_blocker_impact(task_id: str) -> BlockerImpact:
    """Get blocker impact across all namespaces."""
    namespace, id = parse_task_id(task_id)  # e.g., "PLAT", "0023"
    
    # Find all tasks (in any namespace) that depend on this task
    blocked_tasks = []
    for ns in all_namespaces():
        blocked_tasks.extend(
            find_tasks_with_dependency(f"TASK-{namespace}-{id}", ns)
        )
    
    return compute_impact(blocked_tasks)
```

**Example Report**:
```
=== CROSS-NAMESPACE BLOCKER IMPACT ===
Blocker: PLAT-0023 (Database Migration)

Blocked Tasks:
  FW-0042 (Fireweed): User Authentication - blocked 3 days
  FW-0049 (Fireweed): Payment Integration - blocked 1 day
  MOBILE-0015 (Mobile): Login Screen - blocked 2 days

Total Impact: 6 days across 3 namespaces
```

## Namespace-Scoped Operations

### Query Operations

**Default: Scoped to current namespace**
```bash
# Get tasks in FW namespace
task-cli --namespace FW query blocked

# Get WIP for FW namespace
task-cli --namespace FW query wip
```

**Explicit: System-wide across all namespaces**
```bash
# Get all blocked tasks across all namespaces
task-cli --all-namespaces query blocked

# Get system-wide WIP
task-cli --all-namespaces query wip
```

### Reports

**Namespace-Specific Reports**:
```bash
# Weekly flow report for FW namespace
task-cli --namespace FW report weekly-flow

# Blockers report for PLAT namespace
task-cli --namespace PLAT report blockers
```

**System-Wide Reports**:
```bash
# Aggregated weekly flow across all namespaces
task-cli --all-namespaces report weekly-flow

# Cross-namespace blocker impact
task-cli --all-namespaces report blockers
```

### WIP Limits

**Per-Namespace WIP Limits**:
```yaml
# wip.policy.md (namespace-aware)
wip_limits:
  current:
    system_wide: 15       # Total across all namespaces
    per_namespace:
      FW: 5               # Max 5 in current for FW
      PLAT: 5             # Max 5 in current for PLAT
      MOBILE: 5           # Max 5 in current for MOBILE
```

**Enforcement**:
```python
def check_wip_limit(namespace: str, stage: str) -> bool:
    """Check if namespace can add task to stage."""
    current_wip = get_wip(namespace, stage)
    limit = get_wip_limit(namespace, stage)
    return current_wip < limit
```

## Cycle Detection Across Namespaces

**Challenge**: Circular dependencies can span namespaces

**Example**:
```
FW-0042 depends on PLAT-0023
PLAT-0023 depends on MOBILE-0015
MOBILE-0015 depends on FW-0042   # CYCLE!
```

**Detection Algorithm** (extends `WF-DETECT-CYCLES`):
```python
def detect_cycle_across_namespaces(task_id: str) -> bool:
    """Detect cycles across all namespaces."""
    visited = set()
    stack = [task_id]
    
    while stack:
        current = stack.pop()
        if current in visited:
            return True  # Cycle detected
        
        visited.add(current)
        
        # Get dependencies (may be cross-namespace)
        task = get_task(current)  # Handles any namespace
        for dep_id in task.depends_on:
            stack.append(dep_id)
    
    return False
```

**Integration**:
- Run cycle detection before adding cross-namespace dependencies
- Validation fails if cycle would be created
- Log `cycle_detected` event with namespace path

## Namespace Indexing

### Root Index Format

**Location**: `.work/tasks/INDEX.md`

**Format** (adds Namespace column):
```markdown
| Namespace | ID | Name | DRI | Stage | Status | Flags | Dependencies |
|-----------|----|----|-----|-------|--------|-------|--------------|
| FW | 0001 | Feature X | @alice | current | active | - | - |
| FW | 0002 | Feature Y | @bob | backlog | active | - | TASK-FW-0001 |
| PLAT | 0023 | Migration | @charlie | current | active | - | - |
| MOBILE | 0015 | Login UI | @diana | current | blocked | blocked(PLAT-0023) | TASK-PLAT-0023 |
```

### Stage Index Format

**Location**: `.work/tasks/{stage}/INDEX.md`

**Format** (includes namespace):
```markdown
| Namespace | ID | Name | DRI | Status | Flags | Dependencies |
|-----------|----|----|-----|--------|-------|--------------|
| FW | 0003 | API Endpoint | @alice | active | - | - |
| MOBILE | 0013 | UI Component | @diana | active | - | - |
```

## Task ID Reservation

### Per-Namespace Counters

**Location**: 
- Option 1 (unified): `.work/tasks/ID_COUNTER.{namespace}.txt`
- Option 2 (per-namespace): `.work/tasks/{namespace}/ID_COUNTER.txt`

**Content**:
```
43
```
(Next available ID for this namespace)

**Reservation Process** (WF1 pre-step):
```python
def reserve_task_id(namespace: str) -> str:
    """Reserve next task ID for namespace."""
    counter_file = get_counter_file(namespace)
    
    # Read current counter
    with open(counter_file, "r") as f:
        current = int(f.read().strip())
    
    # Increment and write back
    with open(counter_file, "w") as f:
        f.write(str(current + 1))
    
    # Return formatted ID
    return f"{namespace}-{current:04d}"  # e.g., "FW-0043"
```

## Namespace Governance

### Namespace Creation

**Process**:
1. Define namespace in `NAMESPACES.yaml`
2. Create folder structure (if using per-namespace option)
3. Initialize `ID_COUNTER.txt` starting at `1`
4. Create initial `INDEX.md` files
5. Document namespace purpose and team ownership

**Example**:
```yaml
# Add to NAMESPACES.yaml
API:
  full_name: "API Services"
  description: "Backend API development and maintenance"
  team: "API Team"
  created: "2025-11-01"
  id_counter_file: ".work/tasks/api/ID_COUNTER.txt"
```

### Namespace Permissions

**Access Control**:
- Read: View tasks in namespace
- Write: Create/modify tasks in namespace
- Admin: Manage namespace configuration

**Example** (team-based access):
```yaml
# permissions.yaml
namespaces:
  FW:
    read: ["product-eng", "leadership"]
    write: ["product-eng"]
    admin: ["product-leads"]
  
  PLAT:
    read: ["all"]
    write: ["platform-eng"]
    admin: ["platform-leads"]
```

### Namespace Archival

**When to archive namespace**:
- Team disbanded
- Product sunset
- Namespace consolidation

**Process**:
1. Close all active tasks or reassign to other namespaces
2. Move namespace folder to `.work/tasks/.archived_namespaces/{namespace}/`
3. Update `NAMESPACES.yaml` with `status: archived`
4. Update root `INDEX.md` to exclude archived namespace

## Migration Strategies

### From Single Namespace to Multi-Namespace

**Scenario**: Start with tasks numbered 0001-0250, split into two teams

**Strategy**:
1. Create new namespaces (FW, PLAT)
2. Categorize existing tasks by team
3. Rename task IDs (0001 → FW-0001 or PLAT-0001)
4. Update all dependency references
5. Update indexes

**Migration Script**:
```python
def migrate_to_namespaces(task_mapping: Dict[str, str]):
    """
    Migrate tasks to namespaces.
    
    Args:
        task_mapping: {"0001": "FW", "0002": "PLAT", ...}
    """
    for old_id, namespace in task_mapping.items():
        new_id = f"{namespace}-{old_id}"
        
        # Rename task folder
        rename_task_folder(old_id, new_id)
        
        # Update frontmatter task_id
        update_task_frontmatter(new_id, "task_id", new_id)
        
        # Update all dependency references
        update_all_dependencies(old_id, new_id)
        
        # Update indexes
        update_indexes(old_id, new_id, namespace)
```

### From Per-Namespace to Unified Structure

**Scenario**: Consolidate multiple namespace folders into unified structure

**Strategy**:
1. Create unified stage folders
2. Move task folders to unified structure
3. Update paths in scripts/tools
4. Update indexes to unified format

## Implementation Notes

### For Human Execution

**Minimal Namespace Support**:
- Start with single namespace (no prefix needed)
- Add namespaces as teams grow
- Manual ID prefix when creating tasks in new namespaces

**Tools**: Simple prefix convention, no tooling required

### For Automation

**Namespace-Aware CLI**:
```bash
# Set default namespace
task-cli config set namespace FW

# Create task in current namespace
task-cli create "New feature"  # Creates FW-XXXX

# Create task in specific namespace
task-cli create --namespace PLAT "Infrastructure work"  # Creates PLAT-YYYY

# Query across namespaces
task-cli query blocked --all-namespaces
```

**Namespace Context**:
```python
class TaskContext:
    def __init__(self, default_namespace: str = None):
        self.default_namespace = default_namespace or detect_namespace()
    
    def resolve_task_id(self, task_ref: str) -> str:
        """Resolve task reference to full ID."""
        if "-" in task_ref:
            return f"TASK-{task_ref}"  # Already fully qualified
        else:
            return f"TASK-{self.default_namespace}-{task_ref}"
```

## Example: Multi-Team Scenario

### Organization Structure

**Teams**:
- Product Engineering (FW namespace)
- Platform Engineering (PLAT namespace)
- Mobile Engineering (MOBILE namespace)

### Sample Tasks

**FW-0042**: User Authentication Feature
```yaml
task_id: "FW-0042"
task_name: "User Authentication"
depends_on:
  - "TASK-PLAT-0023"   # Database migration
  - "TASK-FW-0010"     # API framework
```

**PLAT-0023**: Database Schema Migration
```yaml
task_id: "PLAT-0023"
task_name: "Auth Schema Migration"
depends_on: []
blocks:
  - "TASK-FW-0042"
  - "TASK-MOBILE-0015"
```

**MOBILE-0015**: Login Screen UI
```yaml
task_id: "MOBILE-0015"
task_name: "Login Screen"
depends_on:
  - "TASK-FW-0042"     # Backend auth API
  - "TASK-PLAT-0023"   # Database ready
```

### Dependency Graph

```
PLAT-0023 (Database Migration)
    ↓
FW-0042 (User Auth API)
    ↓
MOBILE-0015 (Login UI)
```

### Blocker Scenario

If `PLAT-0023` is blocked:
- `FW-0042` is transitively blocked (depends on PLAT-0023)
- `MOBILE-0015` is transitively blocked (depends on both)

**Blocker Report**:
```
Top Blocker: PLAT-0023 (Platform namespace)
  Blocking 2 tasks directly:
    - FW-0042 (Fireweed)
    - MOBILE-0015 (Mobile)
  
  Total Impact: 5 days across 3 namespaces
```

## References

- Query interface: `query.spec.md`
- Metrics schema: `metrics.schema.md`
- WIP policy: `wip.policy.md`
- Graph operations: `workflows/graph-operations.workflow.md`
- Task creation: `inbox/inbox.workflow.md` (WF1)
- Validation: `workflows/validation.workflow.md`
