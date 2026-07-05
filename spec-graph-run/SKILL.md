---
name: spec-graph-run
description: Run or continue a spec-graph session. Auto-selects the latest running session if no ID is given. Shows current task status and next steps. Use when resuming work or checking what to do next.
license: MIT
compatibility: Requires spec-graph CLI (v3.1+) installed.
metadata:
  author: spec-graph
  version: "1.0"
---

Run or continue a spec-graph workflow session.

---

## Orchestrates

| CLI Command | Purpose |
|-------------|---------|
| `spec-graph run` | Continue the latest running session |
| `spec-graph run --session <id>` | Continue a specific session |
| `spec-graph run --auto-next` | Auto-start the next runnable task |

---

## Steps

### 1. Run latest session

```bash
spec-graph run
```

Automatically:
- Finds the most recently updated running session
- Shows session status (stage, progress, blockers)
- Identifies what to resume:
  - If a task is running → shows "Resume task: <id>"
  - If no task running but runnable tasks exist → shows "Next task: <id>"
  - If no runnable tasks → shows "No runnable tasks yet"

### 2. Run specific session

```bash
spec-graph run --session <session-id>
```

Target a specific session instead of auto-selecting.

### 3. Auto-start next task

```bash
spec-graph run --auto-next
```

If no task is currently running:
- Automatically starts the first runnable task
- Equivalent to running `spec-graph task start <next-id>` after seeing the suggestion

### 4. JSON output

```bash
spec-graph run --json
```

Returns structured data including:
```json
{
  "sessionId": "...",
  "intent": "...",
  "stage": "implement",
  "state": "running",
  "progress": { ... },
  "resumeTask": "user-model",
  "action": "resume",
  "taskStatus": {
    "completed": ["infra-setup"],
    "running": ["user-model"],
    "runnable": ["auth-service"],
    "pending": ["api-endpoints", "..."]
  }
}
```

---

## Output Format

### When a task is running:
```
▶ Resume task: user-model
  Continue executing this task.

Task status:
  ✓ Completed (1): infra-setup
  ▶ Running (1): user-model
  → Runnable (2): auth-service, api-endpoints
  ○ Pending (5)

Next steps:
  Dispatch sub-agent for task: spec-graph dispatch --session <id> --task user-model --json
  When task completes: spec-graph task complete user-model --session <id>
```

### When no task is running:
```
→ Next task: user-model
  Run: spec-graph task start user-model --session <id>
  Or:  spec-graph run --session <id> --auto-next
```

### When session is blocked:
```
Blockers:
  - Gate failed: proposal-exists

Resolve blockers before continuing.
```

---

## Session Selection Logic

When no `--session` is specified:
1. Reads `.spec-graph/sessions/sessions.csv`
2. Filters for sessions with `state=running`
3. Sorts by `updated_at` descending
4. Returns the most recently updated session

If no running sessions exist, suggests starting one with `spec-graph plan "<intent>"`.

---

## Edge Cases

- **No running sessions**: Error with suggestion to create one
- **Session not found**: Error if specified session ID doesn't exist
- **Session completed**: Shows "✓ Session already completed"
- **Session blocked**: Shows blockers and suggests resolution
- **No runnable tasks**: Shows "No runnable tasks yet (dependencies pending)"

---

## Integration with Workflow

This skill is the **primary entry point** for continuing work:

1. **Resume work**: `spec-graph run` to see what's next
2. **Start task**: `spec-graph task start <id>` or `spec-graph run --auto-next`
3. **Execute**: Dispatch sub-agent or do manual work
4. **Complete**: `spec-graph task complete <id>`
5. **Loop**: Back to step 1

Use `spec-graph status` for a more detailed view of the session state.
