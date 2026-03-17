# Troubleshooting AO

## Daemon Won't Start

### "daemon already running"
```bash
ao daemon health    # check if actually alive
ao daemon stop      # try graceful stop
# If that fails:
kill $(cat ~/.ao/<repo-scope>/daemon/daemon.lock)
ao daemon start --autonomous ...
```

### Daemon Crashes Immediately
Check log: `tail -50 ~/.ao/<repo-scope>/daemon/daemon.log`

Common causes:
- **Disk full**: worktrees and build artifacts accumulate. Run `cargo clean` or add a cleanup phase.
- **Lock contention**: another process holds the daemon lock.
- **Config error**: invalid `custom.yaml` — check YAML syntax.

## Workflows Fail Immediately (Cancelled in <5 seconds)

### CLAUDECODE Environment Variable
When the daemon is started from inside Claude Code, it inherits session env vars that prevent `claude` CLI from launching.

**Fix**: Build from latest (commit 47d0d192+) which strips these at spawn points. Or:
```bash
env -u CLAUDECODE -u CLAUDE_CODE_SESSION_ACCESS_TOKEN ao daemon start --autonomous ...
```

### Pool Exhaustion
If `pool_size` is too small, cron workflows get cancelled when they can't get a pool slot.

**Fix**: Increase pool_size. Rule of thumb: `pool_size >= concurrent_crons + 2`.
```bash
ao daemon start --autonomous --pool-size 5 ...
```

### "failed to connect runner"
The agent-runner process isn't responding. Usually means it crashed.
```bash
pgrep -f agent-runner    # check if alive
ao daemon stop && ao daemon start --autonomous ...    # restart
```

## Workflows Fail at Implementation Phase

### "no changes were detected"
The agent didn't write any code. Check the task description — it might be too vague.

**Fix**: Update the task with more specific requirements:
```bash
ao task update --id TASK-XXX --description "Specific implementation details..."
```

### "missing required field 'commit_message'"
The implementation phase contract requires a commit. The agent either didn't commit or the output contract wasn't satisfied.

**Fix**: This is usually a prompt quality issue. Ensure the agent prompt says to commit changes.

## Queue Issues

### Stale Assigned Entries
Entries stuck as `assigned` with no running workflow. Happens when daemon restarts mid-workflow.

```bash
ao queue list              # identify stale entries
ao queue drop --subject-id TASK-XXX    # remove them
```

### Queue Not Filling
The work-planner cron should enqueue ready tasks every 5 minutes. If queue stays empty:

1. Check cron status: look at schedule-state.json
2. Check if work-planner workflow succeeds: `ao workflow list --limit 5`
3. Check if there are ready tasks: `ao task list --status ready`
4. Check if tasks have `consecutive_dispatch_failures > 3` (these are skipped)

## Task State Issues

### Tasks Stuck as "blocked"
```bash
ao task list --status blocked
ao task get --id TASK-XXX    # check blocked_reason
ao task status --id TASK-XXX --status ready    # force unblock
```

### Tasks Stuck as "in-progress"
No active workflow but task is still in-progress:
```bash
ao task status --id TASK-XXX --status ready    # reset to ready
```

The task-reconciler cron should catch these automatically.

## PR Issues

### PRs Not Getting Merged
Check if pr-reviewer cron is running and completing:
```bash
# Check schedule state
cat ~/.ao/<repo-scope>/state/schedule-state.json

# Check recent pr-reviewer workflows
ao workflow list --limit 5
```

If pr-reviewer runs but only COMMENTs (never APPROVEs), update the agent prompt in `custom.yaml` to be more decisive about approving.

### Merge Conflicts
When multiple task branches diverge from main:
```bash
gh pr list --state open --json number,mergeable
```

The pr-reviewer should skip conflicted PRs. Rebase manually or create a task to resolve.

## Process Leaks

### Too Many agent-runner Processes
```bash
pgrep -f agent-runner | wc -l
# If > 10, clean up:
pkill -f agent-runner
ao daemon stop
ao daemon start --autonomous ...
```

## State Location Reference

```
~/.ao/<repo-scope>/
├── config/          # Compiled workflow config
├── daemon/          # PID, lock, logs, pm-config
├── scheduler/       # dispatch-queue.json
├── state/           # tasks, schedule-state, errors
├── runs/            # Workflow run output
├── worktrees/       # Git worktrees for task branches
└── workflow-state/  # Per-workflow state files
```

Project local:
```
.ao/
├── config.json      # Project config
└── workflows/       # Custom YAML overlays
    └── custom.yaml
```
