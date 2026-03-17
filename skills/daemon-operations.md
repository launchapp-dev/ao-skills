# Daemon Operations

The AO daemon is a background process that dispatches workflows from the queue, manages agent processes, and runs cron schedules.

## Starting the Daemon

### Basic Start
```bash
ao daemon start
```

### Autonomous Mode (recommended)
```bash
ao daemon start --autonomous --auto-run-ready true --pool-size 5 --interval-secs 10
```

Options:
- `--autonomous` — fork to background, detach from terminal
- `--auto-run-ready true` — automatically dispatch ready tasks from queue
- `--pool-size N` — max concurrent agent workflows (default: 2)
- `--interval-secs N` — tick interval in seconds (default: 15)
- `--auto-merge true` — auto-merge approved PRs
- `--auto-commit-before-merge true` — commit worktree changes before merge

### Pool Size Guidance
- **pool_size=2**: minimal, may starve cron workflows
- **pool_size=5**: good default — 3 crons + 2 task workflows
- **pool_size=8**: heavy workload, needs sufficient API quota

Cron workflows (work-planner, reconciler, pr-reviewer) each take one pool slot when running. If pool_size < number of simultaneous crons, some will be cancelled.

## Monitoring

### Health Check
```bash
ao daemon health
```

Returns: status, active_agents, pool_size, pool_utilization, runner status, queued_tasks.

### Events
```bash
ao daemon events --limit 50
```

Event types: `health`, `queue`, `workflow`, `task-state-change`.

### Logs
```bash
ao daemon logs --limit 100
```

Log file location: `~/.ao/<repo-scope>/daemon/daemon.log`

### Status
```bash
ao daemon status
```

## Stopping

```bash
ao daemon stop
```

Graceful shutdown with 60-second timeout. Running workflows are cancelled.

## MCP Tools

| Tool | Purpose |
|------|---------|
| `ao.daemon.start` | Start the daemon |
| `ao.daemon.stop` | Stop the daemon |
| `ao.daemon.status` | Basic running/stopped check |
| `ao.daemon.health` | Detailed health metrics |
| `ao.daemon.events` | Recent daemon events |
| `ao.daemon.logs` | Read daemon log |
| `ao.daemon.agents` | List active agent processes |
| `ao.daemon.config` | Read daemon config |
| `ao.daemon.config-set` | Update daemon config |
| `ao.daemon.pause` | Pause dispatch |
| `ao.daemon.resume` | Resume dispatch |

## Daemon Architecture

```
ao daemon start
  └─ daemon process (PID in daemon.lock)
       ├─ tick loop (every interval_secs)
       │   ├─ process_due_schedules() — fire cron workflows
       │   ├─ capture_snapshot() — load task/queue/workflow state
       │   ├─ dispatch_ready() — pick tasks from queue, spawn workflows
       │   └─ reconcile() — clean zombie workflows, update state
       └─ runner process (agent-runner binary)
            └─ spawns CLI tools (claude, codex, gemini) per phase
```

## Common Issues

### CLAUDECODE Environment Variable
When starting the daemon from inside Claude Code, the `CLAUDECODE` and `CLAUDE_CODE_SESSION_ACCESS_TOKEN` env vars are inherited. These cause spawned `claude` CLI processes to fail.

**Fix**: The daemon strips these at spawn points automatically (as of commit 47d0d192). If running an older build, use:
```bash
env -u CLAUDECODE -u CLAUDE_CODE_SESSION_ACCESS_TOKEN ao daemon start --autonomous ...
```

### Daemon Crashes
Check the log:
```bash
tail -100 ~/.ao/<repo-scope>/daemon/daemon.log
```

Common causes:
- Disk full (worktrees accumulate)
- Too many agent-runner processes (process leak)
- Lock file contention

### Stale Lock
If the daemon won't start ("daemon already running"):
```bash
# Check if actually running
ps -p $(cat ~/.ao/<repo-scope>/daemon/daemon.lock)
# If not running, the lock is stale — daemon start will clean it up
```

### Process Leak
Agent-runner processes can accumulate across daemon restarts:
```bash
pgrep -f agent-runner | wc -l    # count
pkill -f agent-runner             # clean up
```
