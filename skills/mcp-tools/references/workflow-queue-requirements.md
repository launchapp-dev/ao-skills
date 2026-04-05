# Workflow, Queue, And Requirement Tools

Use this reference when the AO operation is about workflow runtime, workflow definitions, queue dispatch, or requirement state.

## Workflow runtime tools

| Tool | Key parameters |
|------|----------------|
| `ao.workflow.run` | `task_id`, `requirement_id`, `title`, `description`, `workflow_ref`, `input_json` |
| `ao.workflow.run-multiple` | `runs[]`, `on_error` |
| `ao.workflow.execute` | `task_id`, `workflow_ref`, `phase`, `model`, `tool`, `phase_timeout_secs`, `input_json` |
| `ao.workflow.get` | `id` |
| `ao.workflow.list` | `status`, `workflow_ref`, `task_id`, `phase_id`, `search`, `sort`, `limit`, `offset`, `max_tokens` |
| `ao.workflow.pause` | `id`, `confirm`, `dry_run` |
| `ao.workflow.cancel` | `id`, `confirm`, `dry_run` |
| `ao.workflow.resume` | `id` |
| `ao.workflow.phase.approve` | `workflow_id`, `phase_id`, `feedback` |

## Workflow decisions and definitions

| Tool | Key parameters |
|------|----------------|
| `ao.workflow.decisions` | `id`, `limit`, `offset`, `max_tokens` |
| `ao.workflow.checkpoints.list` | `id`, `limit`, `offset`, `max_tokens` |
| `ao.workflow.phases.list` | `project_root` |
| `ao.workflow.phases.get` | `phase` |
| `ao.workflow.definitions.list` | `project_root` |
| `ao.workflow.config.get` | `project_root` |
| `ao.workflow.config.validate` | `project_root` |

## Queue tools

| Tool | Key parameters |
|------|----------------|
| `ao.queue.list` | `project_root` |
| `ao.queue.stats` | `project_root` |
| `ao.queue.enqueue` | `task_id`, `requirement_id`, `title`, `description`, `workflow_ref`, `input_json` |
| `ao.queue.reorder` | `subject_ids[]` |
| `ao.queue.hold` | `subject_id` |
| `ao.queue.release` | `subject_id` |
| `ao.queue.drop` | `subject_id`, `project_root` |

## Requirement tools

| Tool | Key parameters |
|------|----------------|
| `ao.requirements.list` | `limit`, `offset`, `max_tokens`, `status` |
| `ao.requirements.get` | `id` |
| `ao.requirements.create` | `title`, `description`, `priority`, `acceptance_criterion[]` |
| `ao.requirements.update` | `id`, `title`, `description`, `priority`, `status`, `acceptance_criterion[]` |
| `ao.requirements.delete` | `id` |
| `ao.requirements.refine` | `id[]`, `focus`, `use_ai`, `tool`, `model`, `timeout_secs`, `start_runner`, `input_json` |

## Practical patterns

### Enqueue and monitor

1. `ao.queue.enqueue`
2. `ao.queue.list`
3. `ao.workflow.list`
4. `ao.output.monitor` or `ao.output.run`

### Debug a failed workflow

1. `ao.workflow.list`
2. `ao.workflow.get`
3. `ao.output.run` or `ao.output.jsonl`
4. `ao.output.phase-outputs`
5. `ao.task.get`
