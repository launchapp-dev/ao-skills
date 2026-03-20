---
name: getting-started
description: Install AO, create first task, run first workflow — core concepts and project structure
user_invocable: true
auto_invoke: true
---

# Getting Started with AO

AO is a Rust-based agent orchestrator that manages autonomous software development workflows. It coordinates AI agents (Claude, Codex, Gemini) to implement tasks, run tests, create PRs, and review code.

## Prerequisites

- Git
- GitHub CLI (`gh`) authenticated
- At least one AI CLI tool: `claude` (Claude Code), `codex`, or `gemini`

## Install

```bash
# One-line install (macOS/Linux)
curl -fsSL https://raw.githubusercontent.com/launchapp-dev/ao/main/install.sh | bash

# Or install a specific version
AO_VERSION=v0.0.11 curl -fsSL https://raw.githubusercontent.com/launchapp-dev/ao/main/install.sh | bash
```

This installs `ao`, `agent-runner`, and `llm-cli-wrapper` to `~/.local/bin`.

Verify: `ao --version`

## Initialize a Project

```bash
cd /path/to/your/project
ao setup
```

This creates:
- `.ao/config.json` — project-level AO config
- `.ao/workflows/` — directory for custom workflow YAML

## Core Concepts

### Tasks
Units of work. Each task has an ID (TASK-001), title, status, priority, and type.

```bash
ao task create --title "Add user authentication" --priority high --type feature
ao task list --status ready
ao task status --id TASK-001 --status in-progress
```

### Workflows
Multi-phase pipelines that execute tasks. A typical workflow:
1. **requirements** — AI reads the task and plans implementation
2. **implementation** — AI writes code in a git worktree
3. **unit-test** — runs `cargo test` or equivalent
4. **create-pr** — pushes branch and creates GitHub PR

### Daemon
Background process that continuously dispatches workflows from a queue.

```bash
ao daemon start --autonomous --auto-run-ready true --pool-size 3
ao daemon health
ao daemon stop
```

### Queue
Tasks are enqueued for the daemon to dispatch.

```bash
ao queue enqueue --task-id TASK-001
ao queue list
ao queue stats
```

## First Workflow

```bash
# Create a task
ao task create --title "Add health check endpoint" --priority high --type feature

# Enqueue it
ao queue enqueue --task-id TASK-001

# Start the daemon
ao daemon start --autonomous --auto-run-ready true --pool-size 2

# Watch it work
ao daemon health
ao queue list
```

## MCP Integration

AO exposes all operations as MCP tools. When running inside Claude Code or any MCP-aware AI:

```
ao.task.create    — create tasks
ao.task.list      — list tasks by status
ao.queue.enqueue  — add work to the dispatch queue
ao.daemon.health  — check daemon status
ao.workflow.run   — trigger a workflow manually
ao.output.tail    — read agent output
```

## Project Structure

```
your-project/
├── .ao/
│   ├── config.json              # Project config
│   └── workflows/
│       └── custom.yaml          # Custom workflow definitions
└── ~/.ao/<repo-scope>/          # Runtime state (auto-managed)
    ├── config/                  # Compiled workflow + agent config
    ├── daemon/                  # Daemon PID, lock, logs
    ├── scheduler/               # Dispatch queue
    ├── state/                   # Task state, schedule state
    ├── runs/                    # Workflow run output
    └── worktrees/               # Git worktrees for task branches
```
