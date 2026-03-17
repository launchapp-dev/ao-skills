# Workflow Authoring

Workflows are defined in `.ao/workflows/custom.yaml`. They describe agents, phases, workflow pipelines, and cron schedules.

## YAML Structure

```yaml
# Top-level sections
agents:        # Agent profiles with system prompts and model config
phases:        # Phase definitions (agent, command, or gate)
workflows:     # Pipeline definitions chaining phases
schedules:     # Cron schedules for recurring workflows
```

## Agents

Agents are AI profiles with a system prompt and model/tool configuration.

```yaml
agents:
  default:
    model: claude-sonnet-4-6
    tool: claude

  planner:
    system_prompt: |
      You are a planning agent. Break requirements into tasks.
      Use ao.task.create to create tasks with clear acceptance criteria.
    model: claude-sonnet-4-6
    tool: claude

  implementer:
    system_prompt: |
      You implement code changes. Write clean, tested code.
      Commit your changes with descriptive messages.
    model: claude-sonnet-4-6
    tool: claude
```

Fields:
- `system_prompt` — instructions for the agent
- `model` — LLM model ID (claude-sonnet-4-6, gemini-3.1-pro-preview, etc.)
- `tool` — CLI tool to use (claude, codex, gemini)

## Phases

### Agent Phase
Runs an AI agent with a directive.

```yaml
phases:
  implementation:
    mode: agent
    agent: implementer
    directive: "Implement the task requirements. Write code, add tests, commit."
    capabilities:
      mutates_state: true
```

### Command Phase
Runs a shell command.

```yaml
  unit-test:
    mode: command
    directive: "Run the workspace test suite"
    command:
      program: cargo
      args: ["test", "--workspace"]
      timeout_secs: 300
    output_contract:
      kind: phase_result
      fields:
        exit_code:
          type: integer
```

### Gate Phase
Requires human or automated approval to proceed.

```yaml
  review-gate:
    mode: gate
    directive: "Code review must pass before merge"
    gate:
      auto_approve: false
      timeout_secs: 86400
```

## Workflows

Chain phases into a pipeline. Workflows run sequentially through phases.

```yaml
workflows:
  - id: standard
    name: "Standard Task Workflow"
    description: "Requirements → Implement → Test → Create PR"
    phases:
      - requirements
      - implementation
      - unit-test
      - create-pr

  - id: quick-fix
    name: "Quick Fix"
    description: "Skip requirements, go straight to implementation"
    phases:
      - implementation
      - unit-test
      - create-pr
```

### Rework Support
Phases can define rework targets for when a review rejects:

```yaml
  - id: gated
    phases:
      - requirements
      - implementation
      - unit-test
      - code-review:
          on_verdict:
            rework:
              target: implementation
      - create-pr
```

## Cron Schedules

Run workflows on a recurring schedule.

```yaml
schedules:
  - id: work-planner
    cron: "*/5 * * * *"        # Every 5 minutes
    workflow_ref: work-planner
    enabled: true

  - id: pr-reviewer
    cron: "*/5 * * * *"        # Every 5 minutes
    workflow_ref: pr-reviewer
    enabled: true

  - id: system-monitor
    cron: "0 * * * *"          # Every hour
    workflow_ref: system-monitor
    enabled: true
```

Cron expressions use 5-field format: `minute hour day-of-month month day-of-week`

Common patterns:
- `*/5 * * * *` — every 5 minutes
- `*/15 * * * *` — every 15 minutes
- `0 * * * *` — every hour at :00
- `30 */2 * * *` — every 2 hours at :30
- `@daily` — once per day at midnight

## Workflow Variables

Workflows can receive input variables:

```yaml
schedules:
  - id: nightly
    cron: "0 0 * * *"
    workflow_ref: standard
    input: {"scope": "nightly", "full_test": true}
```

## Skills

Phases can reference skills that provide additional context:

```yaml
  pr-review:
    mode: agent
    agent: pr-reviewer
    directive: "Review open PRs"
    skills:
      - code-review
```

## Common Patterns

### Autonomous Loop
```yaml
agents:
  work-planner:
    system_prompt: |
      Scan tasks, enqueue ready work via ao.queue.enqueue.

  reconciler:
    system_prompt: |
      Fix stale state. Unblock tasks. Clean queue.

  pr-reviewer:
    system_prompt: |
      Approve and merge PRs. Mark tasks done.

schedules:
  - {id: work-planner, cron: "*/5 * * * *", workflow_ref: work-planner, enabled: true}
  - {id: reconciler, cron: "*/5 * * * *", workflow_ref: reconciler, enabled: true}
  - {id: pr-reviewer, cron: "*/5 * * * *", workflow_ref: pr-reviewer, enabled: true}
```

### Task Workflow with All Phases
```yaml
workflows:
  - id: standard
    phases:
      - requirements
      - implementation
      - unit-test
      - code-review:
          on_verdict:
            rework:
              target: implementation
      - create-pr
      - cleanup
```
