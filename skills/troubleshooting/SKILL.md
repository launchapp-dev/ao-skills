---
name: troubleshooting
description: Common AO issues and fixes — daemon crashes, workflow failures, queue problems, merge conflicts
user_invocable: true
auto_invoke: true
---

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

## Tasks Marked "Done" But PR Never Merged

The implementation phase may complete successfully, but push/PR phases fail. The task gets marked "done" by the workflow runner even though the code never reached main.

**Detection:** Task status is "done" but `gh pr list --state merged --search "TASK-XXX"` returns nothing.

**Fix in reconciler prompt:**
```
Check for tasks marked "done" that have NO merged PR.
These were marked done prematurely. Set them back to "ready"
and queue rebase-and-retry.
```

## Worktrees Have No node_modules

Command phases (`pnpm build`, `pnpm test`, `pnpm lint`) fail in worktrees because `node_modules` doesn't exist. The implementation agent works fine (uses Claude Code file tools, not the build system), but command phases need deps.

**Fix:** Add `install-deps` command phase before any command that needs `node_modules`:
```yaml
install-deps:
  mode: command
  command:
    program: pnpm
    args: ["install"]
    cwd_mode: task_root
```

## Reviewer Merges PRs With Failing CI

By default, the reviewer agent doesn't check CI status before merging.

**Fix:** Add to reviewer prompt:
```
Before merging, run: gh pr checks <number>
If checks fail, evaluate: is the failure from THIS PR or pre-existing?
- If from this PR: queue rework
- If pre-existing: merge anyway, create a fix task
- If pending: skip, next cycle picks it up
```

## Daemon Doesn't Reload YAML Changes

The daemon reads `.ao/workflows/custom.yaml` at startup and compiles it. Changes to the YAML while the daemon is running are NOT picked up.

**Fix:** Always restart the daemon after editing custom.yaml:
```bash
ao daemon stop
ao daemon start --autonomous --auto-run-ready true --pool-size 3
```

## Tasks Stuck in Infinite Retry Loop

If a task keeps blocking and the planner/reconciler keeps re-enqueuing it, it can waste agent slots forever.

**Detection:** Task version number is very high (>10) and status cycles between ready → blocked.

**Fixes:**
1. Check `ao.output.tail --task-id TASK-XXX` for the actual error
2. Check the worktree: `git -C <worktree_path> log --oneline -3`
3. If the code is committed but push fails: manually push and create PR
4. If the task is fundamentally stuck: cancel it and create a better-scoped replacement
5. Add `consecutive_dispatch_failures > 3` skip rule to the planner prompt

## Parallel Agents Cause Merge Conflicts

Multiple agents running in parallel will conflict on shared files (especially `pnpm-lock.yaml`).

**Mitigations:**
1. Use `rebase-and-retry` workflow — the reviewer queues it for conflicting PRs
2. Reduce `pool_size` to 1 if conflicts are too frequent
3. The rebase agent resolves conflicts intelligently (keeps both sides)

## auto_merge and auto_pr Should Be False

If the daemon's `auto_merge` is `true`, it merges PRs without code review. Set both to `false` and let the pr-review phase handle merging:

```bash
ao daemon config-set --auto-merge false --auto-pr false
```
