# AO Skills

Skills for AI assistants to help developers use the [AO CLI](https://github.com/AudioGenius-ai/ao-cli) — an agent orchestrator for autonomous software development.

## Install

```bash
git clone https://github.com/AudioGenius-ai/ao-skills.git ~/ao-skills
```

### Add to Claude Code
```bash
# Copy the setup command
cp ~/ao-skills/commands/setup-ao.md ~/.claude/commands/

# Or copy all skills as commands
cp ~/ao-skills/skills/*.md ~/.claude/commands/
```

Then run `/setup-ao` in Claude Code to get started.

### Quick Start Prompt
Paste this to any AI assistant:
```
Read all files in ~/ao-skills/skills/ then help me set up AO in this project.
```

## Skills

| Skill | Description |
|-------|-------------|
| [getting-started](skills/getting-started.md) | Install AO, create first task, run first workflow |
| [mcp-setup](skills/mcp-setup.md) | Set up `.mcp.json`, Claude Code permissions, connect AI tools |
| [configuration](skills/configuration.md) | Project config, daemon config, agent runtime, state layout |
| [task-management](skills/task-management.md) | Full task lifecycle via CLI and MCP |
| [workflow-authoring](skills/workflow-authoring.md) | Write custom workflows in YAML |
| [daemon-operations](skills/daemon-operations.md) | Start, monitor, and troubleshoot the daemon |
| [queue-management](skills/queue-management.md) | Dispatch queue operations |
| [mcp-tools](skills/mcp-tools.md) | Complete MCP tool reference with examples |
| [troubleshooting](skills/troubleshooting.md) | Common issues and fixes |

## Usage

These skills teach AI assistants how to help you with AO. When loaded, your AI can:

- Set up `.mcp.json` to connect AO to Claude Code or other AI tools
- Configure AO projects, daemon settings, and agent models
- Set up AO in a new project
- Create and manage tasks with priorities and dependencies
- Write workflow YAML with agents, phases, and cron schedules
- Start and monitor the autonomous daemon
- Debug workflow failures and daemon crashes
- Use all `ao.*` MCP tools correctly
