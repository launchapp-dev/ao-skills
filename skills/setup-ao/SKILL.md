---
name: setup-ao
description: Set up AO in the current project — initialize config, connect MCP, create first workflow
user_invocable: true
auto_invoke: false
---

You are setting up AO — an autonomous agent orchestrator — in this project.

## Step 1: Read the AO skills
Read all files in ~/ao-skills/skills/ to understand how AO works:
- `getting-started/SKILL.md` — install, project structure, core concepts
- `mcp-setup/SKILL.md` — how to create `.mcp.json` and connect AI tools
- `configuration/SKILL.md` — config files, state layout, model routing
- `task-management/SKILL.md` — task lifecycle, CLI and MCP commands
- `workflow-authoring/SKILL.md` — workflow YAML, agents, phases, schedules
- `daemon-operations/SKILL.md` — start/stop daemon, monitoring, troubleshooting
- `queue-management/SKILL.md` — dispatch queue operations
- `mcp-tools/SKILL.md` — current `ao.*` MCP tool reference
- `troubleshooting/SKILL.md` — common failures and fixes

## Step 2: Set up the project
1. Run `ao setup` in the project root to initialize .ao/
2. Create .mcp.json pointing to the ao binary (see mcp-setup skill)
3. Verify MCP connection by calling ao.daemon.status

## Step 3: Create a workflow
Create `.ao/workflows.yaml` or `.ao/workflows/custom.yaml` with:
- At least one agent (model: claude-sonnet-4-6, tool: claude)
- A standard workflow: requirements → implementation → unit-test → create-pr
- Any recurring schedules your project actually needs

## Step 4: Start the daemon
Start with: ao daemon start --autonomous --auto-run-ready true --pool-size 5 --interval-secs 10
Verify with: ao daemon health

## Step 5: Create tasks and let it run
Create tasks via ao task create, enqueue via ao queue enqueue.
The daemon will dispatch workflows based on your configured queue and workflow definitions.

Always check ~/ao-skills/skills/ for reference when unsure about commands, config, or troubleshooting.
