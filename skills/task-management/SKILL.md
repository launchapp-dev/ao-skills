---
name: task-management
description: Full task lifecycle — create, list, update, block/unblock, checklists, bulk ops via CLI and MCP
user_invocable: false
auto_invoke: true
---

# Task Management

## Task Lifecycle

```
backlog → ready → in-progress → done
                ↘ blocked → ready (when unblocked)
                ↘ on-hold → ready (when released)
                ↘ cancelled
```

## CLI Commands

### Create
```bash
ao task create --title "Fix login bug" --priority critical --type bugfix
ao task create --title "Add dark mode" --priority medium --type feature --description "Support system theme preference"
```

Options:
- `--priority`: critical, high, medium, low
- `--type`: feature, bugfix, hotfix, refactor, test, chore, docs, experiment
- `--description`: detailed description
- `--tags`: comma-separated tags

### List and Filter
```bash
ao task list                          # all tasks
ao task list --status ready           # ready tasks only
ao task list --status blocked         # blocked tasks
ao task prioritized                   # sorted by priority
ao task stats                         # aggregate counts by status/priority/type
```

### Get Details
```bash
ao task get --id TASK-001
```

### Update Status
```bash
ao task status --id TASK-001 --status ready
ao task status --id TASK-001 --status in-progress
ao task status --id TASK-001 --status done
ao task status --id TASK-001 --status blocked
ao task status --id TASK-001 --status on-hold
ao task status --id TASK-001 --status cancelled
```

Setting `ready` clears `paused`, `blocked_at`, `blocked_reason`, and `blocked_by`.

### Pause / Resume
```bash
ao task pause --id TASK-001
ao task resume --id TASK-001
```

### Cancel / Reopen
```bash
ao task cancel --id TASK-001
ao task reopen --id TASK-001    # reopens from terminal state back to backlog
```

### Set Priority / Deadline
```bash
ao task set-priority --id TASK-001 --priority critical
ao task set-deadline --id TASK-001 --deadline "2026-04-01T00:00:00Z"
```

### Update Content
```bash
ao task update --id TASK-001 --title "Updated title" --description "New details"
```

### Checklists
```bash
ao task checklist-add --id TASK-001 --item "Add unit tests"
ao task checklist-update --id TASK-001 --index 0 --completed true
```

### Bulk Operations
```bash
ao task bulk-status --ids TASK-001,TASK-002,TASK-003 --status cancelled
```

## MCP Tools

| Tool | Purpose |
|------|---------|
| `ao.task.create` | Create a new task |
| `ao.task.get` | Get task details by ID |
| `ao.task.list` | List tasks, filter by status |
| `ao.task.update` | Update task title/description |
| `ao.task.delete` | Delete a task |
| `ao.task.assign` | Assign a task to an agent or human |
| `ao.task.status` | Change task status |
| `ao.task.stats` | Aggregate task metrics |
| `ao.task.prioritized` | Get tasks sorted by priority |
| `ao.task.next` | Get the next recommended task to work on |
| `ao.task.checklist-add` | Add a checklist item |
| `ao.task.checklist-update` | Mark checklist item complete/incomplete |
| `ao.task.bulk-status` | Update multiple task statuses at once |
| `ao.task.bulk-update` | Bulk update multiple tasks |
| `ao.task.pause` | Pause a task |
| `ao.task.resume` | Resume a paused task |
| `ao.task.cancel` | Cancel a task |
| `ao.task.set-priority` | Set task priority |
| `ao.task.set-deadline` | Set or clear task deadline |
| `ao.task.history` | View task dispatch history |

### MCP Examples

```json
// Create a task
{ "title": "Add rate limiting", "priority": "high", "type": "feature" }

// List ready tasks
{ "status": "ready" }

// Update status
{ "id": "TASK-042", "status": "done" }
```

## Patterns

### Promoting from Backlog
Tasks start as `backlog`. Promote to `ready` when dependencies are met:
```bash
ao task status --id TASK-005 --status ready
```

### Blocking and Unblocking
When a task can't proceed:
```bash
ao task status --id TASK-005 --status blocked
# Later, when the blocker is resolved:
ao task status --id TASK-005 --status ready
```

### Task Dependencies
Tasks can reference dependencies in their description or via `blocked_by`. The work-planner cron checks these before enqueuing.

### Workflow Integration
When the daemon picks up a task from the queue, it:
1. Creates a git worktree (`ao/task-XXX` branch)
2. Runs the workflow phases (requirements → implementation → test → create-pr)
3. On success, the task PR gets reviewed by the pr-reviewer cron
4. On merge, the reconciler marks the task as `done`
