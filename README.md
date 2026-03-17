# AO Skills

Skills for AI assistants to help developers use the [AO CLI](https://github.com/AudioGenius-ai/ao-cli) — an agent orchestrator for autonomous software development.

## Install

```bash
# Claude Code
claude mcp add ao-skills -- cat ~/ao-skills/skills.json

# Or add individual skills to .claude/commands/
cp ~/ao-skills/skills/*.md ~/.claude/commands/
```

## Skills

| Skill | Description |
|-------|-------------|
| [getting-started](skills/getting-started.md) | Install AO, create first task, run first workflow |
| [task-management](skills/task-management.md) | Full task lifecycle via CLI and MCP |
| [workflow-authoring](skills/workflow-authoring.md) | Write custom workflows in YAML |
| [daemon-operations](skills/daemon-operations.md) | Start, monitor, and troubleshoot the daemon |
| [queue-management](skills/queue-management.md) | Dispatch queue operations |
| [mcp-tools](skills/mcp-tools.md) | Complete MCP tool reference with examples |
| [troubleshooting](skills/troubleshooting.md) | Common issues and fixes |

## Usage

These skills teach AI assistants how to help you with AO. When loaded, your AI can:

- Set up AO in a new project
- Create and manage tasks with priorities and dependencies
- Write workflow YAML with agents, phases, and cron schedules
- Start and monitor the autonomous daemon
- Debug workflow failures and daemon crashes
- Use all `ao.*` MCP tools correctly
