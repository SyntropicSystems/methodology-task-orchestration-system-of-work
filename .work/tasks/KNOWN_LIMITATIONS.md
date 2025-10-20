---
type: documentation
last_updated: 2025-10-18
status: living_document
---

# Known Limitations and Manual Workarounds

This document catalogues known limitations in the Task Management System V1.0 that are acceptable for manual human execution but will require tooling/automation to resolve properly.

**Context**: This system is designed as a "perfect human" specification - business logic for manual execution. The limitations below are **implementation concerns** that arise from concurrency, atomicity, and system failures. They have **manual workarounds** documented here and will be **properly solved when building CLI/API tooling**.

---

## 1. ID Counter Race Condition

**Problem**: Multiple people creating tasks simultaneously can reserve the same ID, violating uniqueness.

**Why Deferred**: This is a concurrency problem that only matters with:
- Multiple simultaneous task creators
- Automation creating tasks in parallel
- Small probability with manual coordination

**Manual Workaround**:

### Protocol
1. **Before creating a task**: Announce in team chat: "Creating a task" (reserves ID_COUNTER)
2. Execute WF1 completely without interruption
3. **After creating task**: Announce: "Task TASK-XXXX created" (releases ID_COUNTER)
4. Only one person creates tasks at a time

### If Collision Occurs
1. WF0 (Index Sanity Check) will detect duplicate IDs
2. Manually resolve:
   - Choose which task keeps the original ID (earliest by ledger timestamp)
   - Rename other task's folder and files to next available ID
   - Update all references in indexes
   - Log collision resolution in both task ledgers
3. Update ID_COUNTER.txt to highest ID + 1

### If ID_COUNTER.txt Corrupted
Execute **WF-ID-RECOVERY** (manual):
1. Scan all TASK-* folders across all stages
2. Extract numeric IDs from folder names
3. Find maximum ID: `max_id`
4. Set ID_COUNTER.txt to `max_id + 1`
5. Log recovery: `{"ts":"ISO8601","user":"system","event":"id_counter_recovered","desc":"Rebuilt from max_id=XXXX"}`

**Tooling Solution**: CLI with file locking, atomic read-increment-write, or API with database counter.

---

## 2. Atomic File Operations / Transaction Model

**Problem**: Workflows involve multiple file operations (move folder, update 3 indexes, update frontmatter, append to ledger). If interrupted mid-workflow, system enters inconsistent state with no automatic rollback.

**Why Deferred**: This is an atomicity problem requiring:
- Transaction logs
- Automatic rollback
- Crash recovery

**Manual Workaround**:

### Prevention
1. **Complete workflows in one sitting** - don't stop mid-workflow
2. **If interrupted**, log in task ledger immediately:
   ```json
   {"ts":"ISO8601","user":"user_id","event":"workflow_interrupted","desc":"WF4 interrupted at step 3, will retry"}
   ```
3. Note exact interruption point

### Detection
1. Run **WF0** (Index Sanity Check) periodically (weekly recommended)
2. WF0 will detect:
   - Tasks in folder but not in indexes (orphaned)
   - Tasks in indexes but folder missing (phantom)
   - Stage mismatch between folder and frontmatter
   - Index status mismatch

### Recovery
When WF0 detects inconsistency:

**For Orphaned Tasks** (folder exists, no index entries):
1. Read task ledger to determine intended state
2. Locate last successful event
3. Complete the interrupted workflow OR rollback to previous stage
4. Update all indexes accordingly
5. Log repair: `{"ts":"ISO8601","user":"user_id","event":"orphan_repaired","desc":"Completed interrupted WF4"}`

**For Phantom Entries** (index entry, no folder):
1. Check if task was moved or deleted
2. If moved: update indexes to correct location
3. If deleted accidentally: restore from git history
4. If legitimately gone: remove phantom entries
5. Log repair

**For Stage Mismatches**:
1. Use ledger as source of truth
2. Move folder to match ledger's last stage
3. Update frontmatter stage field
4. Update all indexes
5. Log repair

**Tooling Solution**: CLI with git-like staging, rollback commands, automatic repair, transaction logs.

---

## 3. Concurrent Modification Conflicts

**Problem**: Multiple people editing the same task simultaneously can overwrite each other's changes (last writer wins).

**Why Deferred**: This is a locking/coordination problem requiring:
- File locks
- Optimistic concurrency (checksums)
- Automatic conflict detection

**Manual Workaround**:

### Prevention Protocol
1. **Before modifying a task**: Announce in team chat: "Modifying TASK-0042"
2. Check no one else is working on it (check chat history)
3. Proceed with modification
4. **After modification**: Announce: "TASK-0042 modification complete"

### Small Teams
- Natural coordination occurs
- Low probability of conflicts with 2-5 people
- Index conflicts rare with discipline

### If Conflict Detected
1. **Compare versions**: Check timestamps in both files
2. **Use ledger timestamps** to determine chronological order
3. **Merge manually**:
   - Identify which changes were made by each person
   - Combine changes if possible
   - If irreconcilable, use ledger timestamps (earlier change wins)
4. **Document resolution**:
   ```json
   {"ts":"ISO8601","user":"user_id","event":"conflict_resolved","desc":"Merged changes from @user1 and @user2","resolution":"manual_merge"}
   ```
5. Update both task ledgers with conflict resolution

### Git as Backup
- All files are in git
- Can view history: `git log .work/tasks/current/TASK-0042/`
- Can restore previous version if needed
- Can create branch to resolve conflict

**Tooling Solution**: CLI with lock files, web UI with pessimistic locking, API with row-level locks, optimistic concurrency via checksums.

---

## 4. WF0 Performance and Partial Validation

**Problem**: WF0 scans ALL files (expensive at scale). Running before EVERY task movement is impractical. No concept of "partial validation" for localized changes.

**Why Deferred**: This is a performance optimization requiring:
- Incremental validation
- Indexes and caching
- Automatic repair

**Manual Workaround**:

### WF0-LITE (Lightweight Validation)
Execute before each task move (practical version):

**Validates only the task being moved**:
1. Verify task folder exists
2. Verify task has entries in current stage index + root index
3. Verify frontmatter stage matches folder location
4. Verify dependencies exist (if moving to next/current)
5. If all pass, proceed with move

**Time**: ~30 seconds per task vs ~5-10 minutes for full WF0

### Full WF0 Schedule
- Run weekly as batch job (e.g., Friday afternoons)
- Run after major changes (bulk moves, reorganizations)
- Run when suspecting drift

### Small Task Count
- With < 100 tasks, full WF0 is tolerable (~2-3 minutes)
- Run full WF0 before each move if preferred
- Manual system handles this scale well

### If WF0 Fails Mid-Execution
1. Note which rules passed before failure
2. Address the specific issue found
3. Re-run WF0 from start (entire validation)
4. Don't proceed with moves until WF0 passes

**Tooling Solution**: CLI with incremental validation, indexes, caching, automatic repair, parallelized checks.

---

## 5. Corruption Recovery (ID_COUNTER, Ledgers, Files)

**Problem**: File corruption, accidental deletion, or malformed JSON can break the system.

**Why Deferred**: These are disaster recovery scenarios:
- File corruption (rare)
- Accidental deletion (git provides backup)
- Malformed JSON (caught by parser)
- Low probability events

**Manual Workaround**:

### Git History as Primary Recovery
All files are versioned in git:
```bash
# View history of a file
git log .work/tasks/ID_COUNTER.txt

# Restore deleted file
git checkout HEAD~1 -- .work/tasks/current/TASK-0042/

# Restore entire directory
git checkout HEAD~1 -- .work/tasks/current/TASK-0042/

# View specific version
git show HEAD~3:.work/tasks/ID_COUNTER.txt
```

### ID_COUNTER.txt Recovery
See "1. ID Counter Race Condition" → WF-ID-RECOVERY above

### Ledger JSON Validation
```bash
# Check if ledger is valid JSON
jq . .work/tasks/current/TASK-0042/0042.ledger.jsonl

# If malformed, manually fix or restore from git:
git diff HEAD .work/tasks/current/TASK-0042/0042.ledger.jsonl
# Review changes, fix syntax errors
```

### Task Folder Deletion
```bash
# Restore from git
git checkout HEAD -- .work/tasks/current/TASK-0042/

# If committed deletion, go back further
git log --all --full-history -- .work/tasks/current/TASK-0042/
git checkout <commit> -- .work/tasks/current/TASK-0042/
```

### Index Corruption
1. Restore from git: `git checkout HEAD -- .work/tasks/INDEX.md`
2. OR rebuild from task folders:
   - Scan all task folders
   - Read each task's frontmatter
   - Regenerate index entries
   - Log rebuild event

**Tooling Solution**: CLI repair commands, automatic backups, validation on read, checksums.

---

## Summary Table

| Issue | Impact | Probability | Workaround Effort | Automation Priority |
|-------|--------|-------------|-------------------|---------------------|
| ID Counter Race | Duplicate IDs | Low (with coordination) | Low (announce protocol) | High (file locks) |
| Atomic Operations | Inconsistent state | Medium (if interrupted) | Medium (WF0 repair) | High (transactions) |
| Concurrent Edits | Lost changes | Low (small teams) | Low (announce protocol) | Medium (locks) |
| WF0 Performance | Slow validation | Medium (at scale) | Low (WF0-LITE) | Medium (indexes) |
| Corruption | System failure | Very Low (git backup) | Low (git restore) | Low (automatic) |

---

## When to Fix (Automation Roadmap)

### Phase 1: Basic CLI (First 3 Months)
- Atomic operations with rollback
- File locking for ID reservation
- WF0 automation with incremental validation

### Phase 2: Full CLI (3-6 Months)
- Concurrent edit detection
- Automatic conflict resolution
- Performance optimization (indexes, caching)

### Phase 3: API/Web UI (6-12 Months)
- Database-backed state
- Row-level locking
- Real-time conflict detection
- Automatic backups

---

## Philosophy

These limitations are **acceptable trade-offs** for the manual phase because:

1. **Small teams naturally coordinate** - race conditions are rare
2. **Git provides safety net** - all files versioned, recoverable
3. **WF0 detects drift** - periodic validation catches issues
4. **Ledgers provide truth** - complete audit trail enables recovery
5. **Manual workarounds are simple** - announce protocols, git restore

The system is **safe and sound for manual execution** with these documented protocols. Automation will eliminate these coordination burdens, but the business logic remains the same.

---

## Reporting Issues

If you encounter an issue not covered here:
1. Document the scenario in task ledger
2. Note what failed and why
3. Document your recovery steps
4. Update this file with the new scenario
5. Consider if it reveals a gap in business logic vs implementation concern
