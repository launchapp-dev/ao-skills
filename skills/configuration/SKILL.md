---
name: configuration
description: AO project config, daemon config, agent runtime, environment variables, and state layout
user_invocable: false
auto_invoke: true
---

# AO Configuration

AO has three layers of configuration: project config, daemon config, and workflow config.

## Project Config (`.ao/config.json`)

Created by `ao setup`. Lives in your project root, checked into git.

```json
{
  "project_name": "my-project",
  "version": "1"
}
```

## Daemon Config

Stored at `~/.ao/<repo-scope>/daemon/pm-config.json`. Managed via CLI/MCP, not hand-edited.

```json
{
  "auto_merge_enabled": true,
  "auto_pr_enabled": false,
  "auto_commit_before_merge": true,
  "auto_merge_target_branch": "main",
  "auto_merge_no_ff": true,
  "auto_push_remote": "origin",
  "auto_cleanup_worktree_enabled": true,
  "auto_prune_worktrees_after_merge": false
}
```

### Read Config
```bash
ao daemon config
```
MCP: `ao.daemon.config`

### Update Config
```bash
ao daemon config-set --auto-merge true
ao daemon config-set --auto-pr true
ao daemon config-set --auto-commit-before-merge true
```
MCP: `ao.daemon.config-set`

## Workflow Config (`.ao/workflows/custom.yaml`)

Hand-edited YAML. Defines agents, phases, workflows, and cron schedules. See the [workflow-authoring](../workflow-authoring/SKILL.md) skill for full details.

Location: `.ao/workflows/custom.yaml` (checked into git)

The daemon compiles this into runtime config at:
`~/.ao/<repo-scope>/config/workflow-config.v2.json`

## Agent Runtime Config

Controls which AI model/tool each agent profile uses.

Location: `~/.ao/<repo-scope>/config/agent-runtime-config.v2.json`

```json
{
  "agents": {
    "default": {
      "model": "claude-sonnet-4-6",
      "tool": "claude"
    }
  }
}
```

### Model Options
- `claude-sonnet-4-6` — Claude Sonnet (default for implementation)
- `claude-opus-4-6` — Claude Opus (higher quality, slower)
- `gemini-3.1-pro-preview` — Google Gemini (good for research phases)

### Tool Options
- `claude` — Claude Code CLI
- `codex` — OpenAI Codex CLI
- `gemini` — Google Gemini CLI
- `oai-runner` — OpenAI-compatible model runner

## Environment Variables

| Variable | Purpose |
|----------|---------|
| `AO_CONFIG_DIR` | Override global config directory |
| `AO_ALLOW_NON_EDITING_PHASE_TOOL` | Allow non-write-capable tools (e.g., gemini) for all phases |

## State Layout

```
Project root (.ao/) — checked into git:
├── config.json          # Project config
└── workflows/
    └── custom.yaml      # Workflow YAML (agents, phases, schedules)

Scoped runtime (~/.ao/<repo-scope>/) — NOT in git:
├── config/
│   ├── workflow-config.v2.json        # Compiled workflow config
│   ├── agent-runtime-config.v2.json   # Agent model/tool config
│   └── state-machines.v1.json         # State machine definitions
├── daemon/
│   ├── pm-config.json                 # Daemon automation settings
│   ├── daemon.lock                    # PID lock file
│   └── daemon.log                     # Daemon log
├── scheduler/
│   └── dispatch-queue.json            # Queue state
├── state/
│   ├── tasks/                         # Task JSON files
│   ├── schedule-state.json            # Cron schedule run history
│   └── errors.json                    # Error tracking
├── runs/                              # Workflow run output
├── worktrees/                         # Git worktrees for task branches
└── workflow-state/                    # Per-workflow state files
```

## Repo Scope

`<repo-scope>` is derived from the project path: `<project-name>-<hash>`.

Example: `/Users/you/my-project` → `~/.ao/my-project-a1b2c3d4/`

This ensures multiple projects don't collide in runtime state.

## Precedence

For model/tool selection:
1. Phase-level `runtime.model` override (in workflow YAML)
2. Agent profile `model`/`tool` (in agent-runtime-config)
3. Compiled defaults in the AO binary

For workflow resolution:
1. `.ao/workflows/custom.yaml` (project-local overlays)
2. Built-in workflow definitions (compiled into AO)
