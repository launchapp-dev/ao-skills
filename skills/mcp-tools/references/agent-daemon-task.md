# Agent, Daemon, And Task Tools

Use this reference when the AO operation is primarily about agent control, daemon runtime control, or task lifecycle changes.

## Agent control

| Tool | Key parameters |
|------|----------------|
| `ao.agent.run` | `tool`, `model`, `prompt`, `cwd`, `timeout_secs`, `context_json`, `runtime_contract_json`, `detach`, `run_id`, `runner_scope`, `project_root` |
| `ao.agent.control` | `run_id`, `action` (`pause`, `resume`, `terminate`), `runner_scope` |
| `ao.agent.status` | `run_id`, `runner_scope` |

## Daemon management

| Tool | Key parameters |
|------|----------------|
| `ao.daemon.start` | `pool_size` or `max_agents`, `interval_secs`, `auto_run_ready`, `auto_merge`, `auto_pr`, `auto_commit_before_merge`, `auto_prune_worktrees_after_merge`, `startup_cleanup`, `resume_interrupted`, `reconcile_stale`, `stale_threshold_hours`, `max_tasks_per_tick`, `phase_timeout_secs`, `idle_timeout_secs`, `skip_runner`, `autonomous`, `runner_scope`, `project_root` |
| `ao.daemon.stop` | `project_root` |
| `ao.daemon.status` | `project_root` |
| `ao.daemon.health` | `project_root` |
| `ao.daemon.pause` | `project_root` |
| `ao.daemon.resume` | `project_root` |
| `ao.daemon.events` | `limit`, `project_root` |
| `ao.daemon.agents` | `project_root` |
| `ao.daemon.logs` | `limit`, `search`, `project_root` |
| `ao.daemon.config` | `project_root` |
| `ao.daemon.config-set` | `auto_merge`, `auto_pr`, `auto_commit_before_merge`, `auto_prune_worktrees_after_merge`, `auto_run_ready`, `pool_size` or `max_agents`, `interval_secs`, `max_tasks_per_tick`, `stale_threshold_hours`, `phase_timeout_secs`, `idle_timeout_secs`, `notification_config_json`, `notification_config_file`, `clear_notification_config`, `project_root` |

## Task query tools

| Tool | Key parameters |
|------|----------------|
| `ao.task.list` | `status`, `priority`, `task_type`, `assignee_type`, `tag[]`, `risk`, `linked_requirement`, `linked_architecture_entity`, `search`, `limit`, `offset`, `max_tokens` |
| `ao.task.get` | `id` |
| `ao.task.prioritized` | `status`, `priority`, `assignee_type`, `search`, `limit`, `offset`, `max_tokens` |
| `ao.task.next` | `project_root` |
| `ao.task.stats` | `project_root` |
| `ao.task.history` | `id` |

## Task mutation tools

| Tool | Key parameters |
|------|----------------|
| `ao.task.create` | `title`, `description`, `priority`, `task_type`, `linked_requirement[]`, `linked_architecture_entity[]`, `project_root` |
| `ao.task.update` | `id`, `title`, `description`, `priority`, `status`, `assignee`, `linked_architecture_entity[]`, `replace_linked_architecture_entities`, `input_json` |
| `ao.task.delete` | `id`, `confirm`, `dry_run` |
| `ao.task.status` | `id`, `status` |
| `ao.task.assign` | `id`, `assignee`, `assignee_type`, `agent_role`, `model` |
| `ao.task.pause` | `id` |
| `ao.task.resume` | `id` |
| `ao.task.cancel` | `id`, `confirm`, `dry_run` |
| `ao.task.set-priority` | `id`, `priority` |
| `ao.task.set-deadline` | `id`, `deadline` |
| `ao.task.checklist-add` | `id`, `description` |
| `ao.task.checklist-update` | `id`, `item_id`, `completed` |
| `ao.task.bulk-status` | `updates[]`, `on_error` |
| `ao.task.bulk-update` | `updates[]`, `on_error` |

## Practical patterns

### Check system health

1. `ao.daemon.health`
2. `ao.queue.stats`
3. `ao.task.stats`

### Create and dispatch a task

1. `ao.task.create`
2. `ao.queue.enqueue`
3. `ao.daemon.health`
4. `ao.workflow.list`
