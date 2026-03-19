---
name: mcp-tools
description: Complete AO MCP tool reference — all ao.* tools with inputs, patterns, and usage examples
user_invocable: false
auto_invoke: true
---

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
| `ao.task.list` | `{status?, type?, priority?, search?, sort?, limit?}` | List tasks with filters |
| `ao.task.update` | `{id, title?, description?}` | Update task content |
| `ao.task.delete` | `{id}` | Delete a task |
| `ao.task.assign` | `{id, assignee_type?, agent_role?}` | Assign task to agent or human |
| `ao.task.status` | `{id, status}` | Change task status |
| `ao.task.stats` | `{}` | Aggregate metrics |
| `ao.task.prioritized` | `{}` | Tasks sorted by priority |
| `ao.task.next` | `{}` | Next recommended task |
| `ao.task.checklist-add` | `{id, item}` | Add checklist item |
| `ao.task.checklist-update` | `{id, index, completed}` | Toggle checklist |
| `ao.task.bulk-status` | `{ids, status}` | Bulk status update |
| `ao.task.bulk-update` | `{ids, ...fields}` | Bulk update multiple tasks |
| `ao.task.pause` | `{id}` | Pause a task |
| `ao.task.resume` | `{id}` | Resume a paused task |
| `ao.task.cancel` | `{id}` | Cancel a task |
| `ao.task.set-priority` | `{id, priority}` | Set task priority |
| `ao.task.set-deadline` | `{id, deadline?}` | Set or clear deadline |
| `ao.task.history` | `{id}` | Task dispatch history |

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
| `ao.workflow.list` | `{limit?, status?, workflow_ref?, task_id?}` | List workflows |
| `ao.workflow.get` | `{id}` | Workflow details |
| `ao.workflow.decisions` | `{id}` | Get workflow decisions |
| `ao.workflow.run` | `{task_id?, title?, workflow_ref?}` | Start workflow |
| `ao.workflow.run-multiple` | `{...}` | Run multiple workflows |
| `ao.workflow.execute` | `{...}` | Execute workflow |
| `ao.workflow.cancel` | `{id}` | Cancel workflow |
| `ao.workflow.pause` | `{id}` | Pause workflow |
| `ao.workflow.resume` | `{id}` | Resume workflow |
| `ao.workflow.phase.approve` | `{id}` | Approve manual gate |
| `ao.workflow.phases.list` | `{}` | List phase definitions |
| `ao.workflow.phases.get` | `{id}` | Get phase definition |
| `ao.workflow.definitions.list` | `{}` | List workflow definitions |
| `ao.workflow.config.get` | `{}` | Get workflow config |
| `ao.workflow.config.validate` | `{}` | Validate workflow config |
| `ao.workflow.checkpoints.list` | `{id}` | List workflow checkpoints |

## Output Tools

| Tool | Input | Purpose |
|------|-------|---------|
| `ao.output.tail` | `{run_id?, task_id?, limit?}` | Recent output |
| `ao.output.run` | `{run_id}` | Full run output |
| `ao.output.monitor` | `{run_id}` | Monitor running workflow |
| `ao.output.jsonl` | `{run_id}` | JSONL formatted output |
| `ao.output.phase-outputs` | `{run_id}` | Per-phase output |
| `ao.output.artifacts` | `{run_id}` | Run artifacts |

## Requirement Tools

| Tool | Input | Purpose |
|------|-------|---------|
| `ao.requirements.list` | `{status?, priority?, type?, category?}` | List requirements |
| `ao.requirements.get` | `{id}` | Get requirement details |
| `ao.requirements.create` | `{title, description?, type?, priority?}` | Create requirement |
| `ao.requirements.update` | `{id, ...fields}` | Update requirement |
| `ao.requirements.delete` | `{id}` | Delete requirement |
| `ao.requirements.refine` | `{id}` | Refine requirement details |

## Runner Tools

| Tool | Input | Purpose |
|------|-------|---------|
| `ao.runner.health` | `{}` | Runner process health |
| `ao.runner.orphans-detect` | `{}` | Detect orphaned runs |
| `ao.runner.restart-stats` | `{}` | Restart statistics |

## Agent Tools

| Tool | Input | Purpose |
|------|-------|---------|
| `ao.agent.run` | `{...}` | Run agent execution |
| `ao.agent.status` | `{}` | Get agent status |
| `ao.agent.control` | `{action}` | Pause/resume/terminate agent |

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
