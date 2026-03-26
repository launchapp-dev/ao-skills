---
name: queue-management
description: Dispatch queue operations — enqueue, hold, release, drop, reorder, and queue patterns
user_invocable: false
auto_invoke: true
---

# Queue Management

The dispatch queue controls what work the daemon picks up next. Subjects can be tasks, requirements, or custom dispatches.

## Queue Entry States

```
pending → assigned → (removed on completion)
pending → held → pending (released)
any → dropped (via ao queue drop)
```

## CLI Commands

### List Queue
```bash
ao queue list
```

Shows each entry's `subject_id`, status, and selected metadata.

### Queue Stats
```bash
ao queue stats
```

Returns: total, pending, assigned, held counts.

### Enqueue a Task
```bash
ao queue enqueue --task-id TASK-001
ao queue enqueue --task-id TASK-001 --workflow-ref ao.task/quick-fix
```

The daemon picks up pending entries and assigns them to agents.

### Enqueue Other Subjects
```bash
# Requirement
ao queue enqueue --requirement-id REQ-039

# Custom subject
ao queue enqueue --title "Run nightly build" --description "Verify the release branch" --workflow-ref ops
```

### Hold / Release
Temporarily prevent a queued task from being dispatched:
```bash
ao queue hold --subject-id TASK-001
ao queue release --subject-id TASK-001
```

### Drop (Remove)
Remove a queue entry regardless of status:
```bash
ao queue drop --subject-id TASK-001
```

Use this to clean up stale assigned entries that are stuck.

### Reorder
Set dispatch priority order:
```bash
ao queue reorder --subject-id TASK-003 --subject-id TASK-001 --subject-id TASK-002
```

Repeat `--subject-id` in the exact order you want the daemon to consider.

## MCP Tools

| Tool | Purpose |
|------|---------|
| `ao.queue.list` | List all queue entries |
| `ao.queue.stats` | Aggregate queue metrics |
| `ao.queue.enqueue` | Add a dispatch to the queue |
| `ao.queue.hold` | Hold a pending entry |
| `ao.queue.release` | Release a held entry |
| `ao.queue.drop` | Remove an entry (any status) |
| `ao.queue.reorder` | Set dispatch order |

### MCP Examples

```json
// Enqueue a task
{ "task_id": "TASK-042" }

// Enqueue with workflow override
{ "task_id": "TASK-042", "workflow_ref": "ao.task/quick-fix" }

// Drop a stuck entry
{ "subject_id": "TASK-042" }
```

## Patterns

### Queue Draining
When all pending entries are processed, the queue is empty. Refill it by enqueueing tasks directly or by running whatever workflow/schedule in your project is responsible for planning work.

### Stale Assigned Entries
If a workflow completes or fails but the queue entry stays `assigned`, it's stale. The reconciler cron should drop these, or use:
```bash
ao queue drop --subject-id TASK-XXX
```

### Duplicate Prevention
Treat `ao queue enqueue` as safe to retry, but still verify current queue state with `ao queue list` when debugging duplicate work.

### Queue Capacity
The daemon dispatches from the queue up to its configured `pool_size`. If the queue has more pending work than open slots, the rest stay pending until capacity frees up.
