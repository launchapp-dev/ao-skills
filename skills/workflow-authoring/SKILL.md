---
name: workflow-authoring
description: Write custom workflow YAML — agents, phases, pipelines, cron schedules, MCP server connections
user_invocable: true
auto_invoke: true
---

# Workflow Authoring

Project-authored workflow sources live in `.ao/workflows.yaml` and `.ao/workflows/*.yaml`. They describe agents, phases, workflow pipelines, schedules, and MCP server connections.

## Complete YAML Structure

```yaml
# Common top-level sections
tools_allowlist:   # Shell programs command phases can invoke
mcp_servers:       # External MCP servers available to agents
agents:            # Agent profiles with system prompts, model, and MCP bindings
phases:            # Phase definitions
workflows:         # Pipeline definitions chaining phases
schedules:         # Recurring workflows
```

## Tools Allowlist

Controls which programs command phases can run. Also controls Claude Code built-in tools.

```yaml
tools_allowlist:
  - npm
  - npx
  - node
  - pnpm
  - git
  - gh
  - WebSearch    # Claude Code's web search tool
  - WebFetch     # Claude Code's web fetch tool
```

## MCP Servers

Define external MCP servers that agents can connect to. Servers are referenced by name in agent profiles.

```yaml
mcp_servers:
  context7:
    command: npx
    args: ["-y", "@upstash/context7-mcp"]
  package-version:
    command: npx
    args: ["-y", "mcp-package-version"]
  sequential-thinking:
    command: npx
    args: ["-y", "@modelcontextprotocol/server-sequential-thinking"]
  memory:
    command: npx
    args: ["-y", "@modelcontextprotocol/server-memory"]
  github:
    command: npx
    args: ["-y", "@modelcontextprotocol/server-github"]
```

See [mcp-servers-for-agents](../mcp-servers-for-agents/SKILL.md) for details.

## Agents

Agent profiles define the AI model, CLI tool, system prompt, and MCP server access.

```yaml
agents:
  default:
    model: claude-sonnet-4-6
    tool: claude

  implementer:
    system_prompt: |
      You implement code changes. Write clean, type-safe code.
      Commit with descriptive messages. Do not add yourself as author.
    model: claude-sonnet-4-6
    tool: claude
    mcp_servers: ["ao", "context7"]
```

### Agent Profile Fields

| Field | Type | Description |
|-------|------|-------------|
| `description` | string | Agent description |
| `system_prompt` | string | Instructions for the agent |
| `role` | string | Agent role identifier |
| `model` | string | LLM model ID (claude-sonnet-4-6, claude-opus-4-6, gemini-3.1-pro-preview) |
| `tool` | string | CLI tool (claude, codex, gemini, oai-runner) |
| `mcp_servers` | list | MCP server names this agent can access (e.g. ["ao", "context7"]) |
| `web_search` | bool | Enable web search capability |
| `network_access` | bool | Enable network access |
| `extra_args` | list | Extra CLI arguments passed to the tool |
| `fallback_models` | list | Models to try if primary fails |
| `max_attempts` | int | Retry attempts |
| `max_continuations` | int | Max continuations per phase |
| `timeout_secs` | int | Agent timeout |
| `reasoning_effort` | string | Extended thinking level (low, medium, high) |
| `tool_policy` | object | Allow/deny glob patterns per agent |
| `skills` | list | Skill identifiers for additional context |
| `capabilities` | object | Boolean capability flags |
| `project_overrides` | object | Per-project tool/model/args overrides |
| `codex_config_overrides` | object | Codex-specific configuration overrides |

### Model Options
- `claude-sonnet-4-6` — Claude Sonnet (default, good balance)
- `claude-opus-4-6` — Claude Opus (highest quality, slower)
- `gemini-3.1-pro-preview` — Google Gemini (1M context, good for UI/UX)

### Tool Options
- `claude` — Claude Code CLI
- `codex` — OpenAI Codex CLI
- `gemini` — Google Gemini CLI
- `oai-runner` — OpenAI-compatible model runner

## Phases

### Agent Phase
Runs an AI agent with a directive.

```yaml
phases:
  implementation:
    mode: agent
    agent: implementer
    directive: "Implement the task requirements. Write code, commit."
    capabilities:
      mutates_state: true
```

### Command Phase
Runs a shell command. Use for deterministic operations.

```yaml
  push-branch:
    mode: command
    directive: "Push the current branch to origin"
    command:
      program: git
      args: ["push", "-u", "origin", "HEAD"]
      cwd_mode: task_root       # Run in the worktree, not project root
      timeout_secs: 60
    output_contract:
      kind: phase_result
      fields:
        exit_code:
          type: integer
```

#### cwd_mode (important!)

Controls where the command runs:
- `task_root` — the task's git worktree (default — **use this for git/build/test commands**)
- `project_root` — main repo directory
- `path` — custom relative path (requires `cwd_path`)

### Manual Phase
Requires human approval to proceed.

```yaml
  review-gate:
    mode: manual
    directive: "Code review must pass before merge"
    manual:
      instructions: "Review the code changes and approve or reject"
      approval_note_required: false
      timeout_secs: 86400
```

## Workflows

Chain phases into a sequential pipeline.

```yaml
workflows:
  - id: scaffold
    name: "Scaffold Workflow"
    description: "Implement -> Build -> Push -> PR -> CI -> Review"
    phases:
      - implementation
      - install-deps
      - build-check
      - lint-check
      - push-branch
      - create-pr
      - wait-for-ci
      - pr-review:
          on_verdict:
            rework:
              target: implementation
```

Workflow refs are selected with `ao workflow run --workflow-ref <ref>` or `ao queue enqueue --workflow-ref <ref>`.

### Rework Support

Phases can loop back to an earlier phase when a review rejects:

```yaml
  - pr-review:
      on_verdict:
        rework:
          target: implementation
```

**Only use rework on agent phases** (like pr-review). Don't put rework on command phases (build-check, lint-check, wait-for-ci) — it creates infinite loops. Let command phases fail cleanly; the PO/auditor will create a fix task.

### Multiple Workflows

Define workflows for different scenarios:

```yaml
  - id: standard
    phases: [requirements, implementation, install-deps, build-check, lint-check, push-branch, create-pr, wait-for-ci, pr-review]

  - id: scaffold
    phases: [implementation, install-deps, build-check, lint-check, push-branch, create-pr, wait-for-ci, pr-review]

  - id: quick-fix
    phases: [implementation, install-deps, build-check, lint-check, push-branch, create-pr, wait-for-ci, pr-review]

  - id: rework
    description: "Address reviewer feedback"
    phases: [address-review, force-push, pr-review]

  - id: rebase-and-retry
    description: "Rebase conflicting branch"
    phases: [rebase-on-main, force-push, pr-review]
```

## Cron Schedules

Run workflows on a recurring schedule.

```yaml
schedules:
  - id: work-planner
    cron: "*/5 * * * *"
    workflow_ref: work-planner
    enabled: true

  - id: pr-reviewer
    cron: "*/3 * * * *"
    workflow_ref: pr-reviewer
    enabled: true

  - id: task-reconciler
    cron: "2-59/10 * * * *"
    workflow_ref: task-reconciler
    enabled: true
```

Cron expressions use 5-field format: `minute hour day-of-month month day-of-week`

Common patterns:
- `*/5 * * * *` — every 5 minutes
- `*/3 * * * *` — every 3 minutes
- `2-59/10 * * * *` — every 10 minutes starting at :02 (staggered)
- `37 */2 * * *` — every 2 hours at :37

## Validation

Validate and inspect the effective config before restarting the daemon:

```bash
ao workflow config validate
ao workflow config compile
ao workflow definitions list
ao workflow phases list
```
- `0 */3 * * *` — every 3 hours
- `0 9 * * 1` — weekly Monday at 9am

**Stagger cron offsets** to avoid multiple agents competing for pool slots at the same time.

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

## Design Principles

1. **Agent phases for decisions, command phases for execution.** Don't use an agent to run `git push`.
2. **Always `cwd_mode: task_root`** for commands that need to run in the worktree.
3. **Always add `install-deps`** before commands that need `node_modules`.
4. **No rework loops on command phases.** Let them fail; the PO/auditor creates fix tasks.
5. **Only `pr-review` gets rework.** Reviewer feedback is actionable; build failures are not.
6. **Stagger cron schedules.** Don't run all crons at the same offset.
7. **Restart daemon after YAML changes.** The daemon reads YAML once at startup.
8. **Set `auto_merge: false` and `auto_pr: false`** in daemon config. Let workflows handle merging.

## Complete Example

See [workflow-patterns](../workflow-patterns/SKILL.md) for production-ready pipeline examples including QA gates, conflict resolution, CI checks, and stale PR handling.
