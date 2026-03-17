# AO MCP Tools Reference

All AO operations are available as MCP tools. Use these from any MCP-aware AI assistant (Claude Code, etc.).

## Tool Naming Convention

All tools are prefixed with `ao.` and grouped by domain:

```
ao.task.*       — task management
ao.queue.*      — dispatch queue
ao.daemon.*     — daemon control
ao.workflow.*   — workflow execution
ao.output.*     — run output and artifacts
ao.requirements.* — requirement management
ao.runner.*     — runner process management
ao.agent.*      — agent control
```

## Task Tools

| Tool | Input | Purpose |
|------|-------|---------|
| `ao.task.create` | `{title, priority?, type?, description?}` | Create a task |
| `ao.task.get` | `{id}` | Get task details |
| `ao.task.list` | `{status?}` | List tasks by status |
| `ao.task.update` | `{id, title?, description?}` | Update task content |
| `ao.task.status` | `{id, status}` | Change task status |
| `ao.task.stats` | `{}` | Aggregate metrics |
| `ao.task.prioritized` | `{}` | Tasks sorted by priority |
| `ao.task.next` | `{}` | Next recommended task |
| `ao.task.checklist-add` | `{id, item}` | Add checklist item |
| `ao.task.checklist-update` | `{id, index, completed}` | Toggle checklist |
| `ao.task.bulk-status` | `{ids, status}` | Bulk status update |
| `ao.task.cancel` | `{id}` | Cancel a task |
| `ao.task.history` | `{id}` | State change history |

## Queue Tools

| Tool | Input | Purpose |
|------|-------|---------|
| `ao.queue.list` | `{}` | List queue entries |
| `ao.queue.stats` | `{}` | Queue depth metrics |
| `ao.queue.enqueue` | `{task_id?, requirement_id?, title?, workflow_ref?}` | Add to queue |
| `ao.queue.hold` | `{subject_id}` | Hold entry |
| `ao.queue.release` | `{subject_id}` | Release held entry |
| `ao.queue.drop` | `{subject_id}` | Remove entry |
| `ao.queue.reorder` | `{subject_ids: [...]}` | Set order |

## Daemon Tools

| Tool | Input | Purpose |
|------|-------|---------|
| `ao.daemon.start` | `{autonomous?, pool_size?, interval_secs?, auto_run_ready?}` | Start daemon |
| `ao.daemon.stop` | `{}` | Stop daemon |
| `ao.daemon.status` | `{}` | Running/stopped |
| `ao.daemon.health` | `{}` | Detailed health |
| `ao.daemon.events` | `{limit?}` | Recent events |
| `ao.daemon.logs` | `{limit?}` | Read log file |
| `ao.daemon.agents` | `{}` | Active agents |
| `ao.daemon.config` | `{}` | Read config |
| `ao.daemon.config-set` | `{key, value}` | Update config |
| `ao.daemon.pause` | `{}` | Pause dispatch |
| `ao.daemon.resume` | `{}` | Resume dispatch |

## Workflow Tools

| Tool | Input | Purpose |
|------|-------|---------|
| `ao.workflow.list` | `{limit?, status?}` | List workflows |
| `ao.workflow.get` | `{id}` | Workflow details |
| `ao.workflow.run` | `{task_id?, title?, workflow_ref?}` | Start workflow |
| `ao.workflow.cancel` | `{id}` | Cancel workflow |
| `ao.workflow.pause` | `{id}` | Pause workflow |
| `ao.workflow.resume` | `{id}` | Resume workflow |

## Output Tools

| Tool | Input | Purpose |
|------|-------|---------|
| `ao.output.tail` | `{run_id?, task_id?, limit?}` | Recent output |
| `ao.output.run` | `{run_id}` | Full run output |
| `ao.output.artifacts` | `{run_id}` | Run artifacts |

## Common Patterns

### Check System Health
```
1. ao.daemon.health → check status, active_agents, pool_utilization
2. ao.queue.stats → check pending/assigned depth
3. ao.task.stats → check blocked/ready counts
```

### Enqueue and Monitor
```
1. ao.task.create → create the task
2. ao.queue.enqueue → add to queue with task_id
3. ao.daemon.health → verify daemon is running
4. ao.queue.list → watch for assigned status
5. ao.workflow.list → find running workflow
```

### Debug a Failed Workflow
```
1. ao.workflow.list → find failed workflow ID
2. ao.workflow.get → check failure_reason, current_phase
3. ao.output.tail → read agent output for errors
4. ao.task.get → check task state
```

### Clean Up Stuck State
```
1. ao.queue.list → find stale assigned entries
2. ao.queue.drop → remove stuck entries
3. ao.task.list {status: "blocked"} → find blocked tasks
4. ao.task.status → set back to ready
```

## All Tools Accept project_root

Every tool accepts an optional `project_root` parameter. If omitted, AO uses the current working directory's git root.

```json
{ "project_root": "/path/to/project", "task_id": "TASK-001" }
```
