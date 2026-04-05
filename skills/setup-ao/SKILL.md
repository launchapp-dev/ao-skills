---
name: setup-ao
description: Set up AO in the current project - initialize config, connect MCP, create a first workflow, and start the daemon. Use when bootstrapping AO in a repo or fixing an incomplete AO setup.
user_invocable: true
auto_invoke: false
---

You are setting up AO in the current project.

Do not preload the entire `~/ao-skills/skills/` directory. Start with targeted reads:

- Read [getting-started](../getting-started/SKILL.md) first for the core AO mental model.
- Read [mcp-setup](../mcp-setup/SKILL.md) only when creating or fixing `.mcp.json`.
- Read [workflow-authoring](../workflow-authoring/SKILL.md) only when editing `.ao/workflows.yaml` or `.ao/workflows/*.yaml`.
- Read [daemon-operations](../daemon-operations/SKILL.md) only when starting, checking, or debugging the daemon.
- Read [troubleshooting](../troubleshooting/SKILL.md) only if setup fails.
- Use [configuration](../configuration/SKILL.md), [task-management](../task-management/SKILL.md), [queue-management](../queue-management/SKILL.md), and [mcp-tools](../mcp-tools/SKILL.md) as lookup references, not mandatory preload.

## Setup flow

1. Run `ao setup` in the project root to initialize `.ao/`.
2. Create `.mcp.json` pointing to the `ao` binary.
3. Create a minimal workflow file.
4. Start the daemon with conservative defaults.
5. Create one small test task and verify end-to-end execution.

## Minimal workflow

Start with a small workflow instead of a full production pipeline:

```yaml
agents:
  default:
    model: claude-sonnet-4-6
    tool: claude

phases:
  implementation:
    mode: agent
    agent: default
    directive: Implement the task and report what changed.

workflows:
  - id: standard
    name: Standard
    phases: [implementation]
```

Expand it only after the daemon, MCP, and task loop work.

## Daemon startup

Start with:

```bash
ao daemon start --autonomous --auto-run-ready true --pool-size 5 --interval-secs 10
ao daemon health
```

## Verification

- Run `ao daemon status` or the MCP equivalent.
- Create a small task with `ao task create`.
- Enqueue it with `ao queue enqueue`.
- Confirm the daemon picks it up before adding more workflows or schedules.

If something fails, read only the skill that matches the blocker instead of sweeping the whole repo.
