---
name: spec-graph-task
description: Manage tasks within a spec-graph session. Start tasks, mark them complete, and list task status. Automatically updates tasks.md, state.yaml, and sessions.csv. Use when working with individual implementation tasks.
license: MIT
compatibility: Requires spec-graph CLI (v3.1+) installed.
metadata:
  author: spec-graph
  version: "1.0"
---

Manage individual tasks within a spec-graph session.

---

## Orchestrates

| CLI Command | Purpose |
|-------------|---------|
| `spec-graph task list` | List all tasks and their status |
| `spec-graph task start <task-id>` | Mark a task as running |
| `spec-graph task complete <task-id>` | Mark a task as completed |

All commands auto-select the latest running session if `--session` is not specified.

---

## Steps

### 1. List tasks

```bash
spec-graph task list
```

Shows tasks grouped by status:
- ✓ Completed
- ▶ Running
- → Runnable (dependencies satisfied, ready to start)
- ○ Pending (dependencies not yet satisfied)

### 2. Start a task

```bash
spec-graph task start <task-id>
```

Marks the task as running and updates:
- `state.yaml` - adds taskStatus tracking
- `tasks.md` - marks checkbox as `[>]` (running)
- `sessions.csv` - updates running_tasks column

### 3. Complete a task

```bash
spec-graph task complete <task-id>
```

Marks the task as completed and:
- Updates all three files (state.yaml, tasks.md, sessions.csv)
- Computes the next runnable task based on dependency graph
- Returns the next task ID for workflow continuity

### 4. Specify session (optional)

```bash
spec-graph task list --session <session-id>
```

Explicitly target a specific session instead of auto-selecting.

---

## File Synchronization

Task operations automatically update three files:

1. **state.yaml** - `taskStatus` field tracks each task's state
2. **tasks.md** - Checkbox markers (`[ ]` → `[>]` → `[x]`)
3. **sessions.csv** - Four task columns:
   - `completed_tasks` - finished task IDs
   - `running_tasks` - currently executing task IDs
   - `runnable_tasks` - tasks ready to start (dependencies met)
   - `pending_tasks` - tasks waiting on dependencies

---

## Edge Cases

- **No running session**: Command will fail with "No session specified and no running sessions found"
- **Task not in plan**: Error if task-id doesn't exist in the session's plan.order
- **Dependencies not met**: Task won't appear in runnable_tasks until all dependsOn tasks are completed
- **Auto-select ambiguity**: When multiple sessions are running, the most recently updated one is selected

---

## Integration with Workflow

Use this skill during the **implement stage** when working through tasks sequentially or in parallel:

1. Start with `spec-graph task list` to see what's available
2. Pick a runnable task and `spec-graph task start <id>`
3. Execute the task (dispatch sub-agent or manual work)
4. `spec-graph task complete <id>` when done
5. Repeat until all tasks are complete

The `--auto-next` flag on `spec-graph run` can automate task selection.
