---
name: setup-ao
description: Set up AO in the current project — initialize config, connect MCP, create first workflow
user_invocable: true
auto_invoke: false
---

You are setting up AO — an autonomous agent orchestrator — in this project.

## Step 1: Read the AO skills
Read all files in ~/ao-skills/skills/ to understand how AO works:
- getting-started.md — install, project structure, core concepts
- mcp-setup.md — how to create .mcp.json and connect AI tools
- configuration.md — config files, state layout, model routing
- task-management.md — task lifecycle, CLI and MCP commands
- workflow-authoring.md — write custom.yaml with agents, phases, crons
- daemon-operations.md — start/stop daemon, monitoring, troubleshooting
- queue-management.md — dispatch queue operations
- mcp-tools.md — all ao.* MCP tool reference
- troubleshooting.md — common failures and fixes

## Step 2: Set up the project
1. Run `ao setup` in the project root to initialize .ao/
2. Create .mcp.json pointing to the ao binary (see mcp-setup skill)
3. Verify MCP connection by calling ao.daemon.status

## Step 3: Create a workflow
Create .ao/workflows/custom.yaml with:
- At least one agent (model: claude-sonnet-4-6, tool: claude)
- A standard workflow: requirements → implementation → unit-test → create-pr
- Cron schedules: work-planner (5min), task-reconciler (5min), pr-reviewer (5min)

## Step 4: Start the daemon
Start with: ao daemon start --autonomous --auto-run-ready true --pool-size 5 --interval-secs 10
Verify with: ao daemon health

## Step 5: Create tasks and let it run
Create tasks via ao task create, enqueue via ao queue enqueue.
The daemon will dispatch workflows, agents will implement, and PRs will be created and reviewed automatically.

Always check ~/ao-skills/skills/ for reference when unsure about commands, config, or troubleshooting.
