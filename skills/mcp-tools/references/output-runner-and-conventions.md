# Output, Runner, And Conventions

Use this reference when reading AO execution output, checking runner health, or deciding how to shape list and batch requests.

## Output tools

| Tool | Key parameters |
|------|----------------|
| `ao.output.run` | `run_id` |
| `ao.output.tail` | `run_id`, `task_id`, `event_types[]`, `limit` |
| `ao.output.monitor` | `run_id`, `task_id`, `phase_id` |
| `ao.output.jsonl` | `run_id`, `entries` |
| `ao.output.artifacts` | `execution_id` |
| `ao.output.phase-outputs` | `workflow_id`, `phase_id`, `project_root` |

## Runner tools

| Tool | Key parameters |
|------|----------------|
| `ao.runner.health` | `project_root` |
| `ao.runner.orphans-detect` | `project_root` |
| `ao.runner.orphans-cleanup` | `run_id`, `project_root` |
| `ao.runner.restart-stats` | `project_root` |

## Shared conventions

Every MCP tool accepts optional `project_root`.

```json
{ "project_root": "/path/to/project" }
```

## List pagination

List tools use common pagination fields:

| Parameter | Default | Max | Notes |
|-----------|---------|-----|-------|
| `limit` | 25 | 200 | Maximum items to return |
| `offset` | 0 | — | Items to skip |
| `max_tokens` | 3000 | 12000 | Response compaction budget |

List responses use the `ao.mcp.list.result.v1` envelope.

## Batch behavior

Batch tools such as `ao.task.bulk-status`, `ao.task.bulk-update`, and `ao.workflow.run-multiple` accept `on_error`.

| Value | Behavior |
|-------|----------|
| `continue` | Process all items even if some fail |
| `stop` | Stop after the first failure and mark the rest skipped |

Batch responses use the `ao.mcp.batch.result.v1` envelope.

Maximum batch size is 100 items per call.
