# Workflow Patterns — Production-Ready Pipelines

Battle-tested patterns from building a full SaaS monorepo with AO. These patterns address real issues discovered over 150+ autonomous PRs.

## The Scaffold Pipeline (recommended for all new projects)

```yaml
phases:
  implementation:
    mode: agent
    agent: implementer
    directive: "Implement the task requirements. Write code, commit."
    capabilities:
      mutates_state: true

  install-deps:
    mode: command
    directive: "Install dependencies in the worktree"
    command:
      program: pnpm
      args: ["install"]
      cwd_mode: task_root
      timeout_secs: 120

  build-check:
    mode: command
    directive: "Verify the project builds"
    command:
      program: pnpm
      args: ["build"]
      cwd_mode: task_root
      timeout_secs: 300

  lint-check:
    mode: command
    directive: "Run lint"
    command:
      program: pnpm
      args: ["lint"]
      cwd_mode: task_root
      timeout_secs: 120

  push-branch:
    mode: command
    directive: "Push the branch to origin"
    command:
      program: git
      args: ["push", "-u", "origin", "HEAD"]
      cwd_mode: task_root
      timeout_secs: 60

  create-pr:
    mode: command
    directive: "Create a PR"
    command:
      program: gh
      args: ["pr", "create", "--fill", "--base", "main"]
      cwd_mode: task_root
      timeout_secs: 60

  wait-for-ci:
    mode: command
    directive: "Wait for CI checks"
    command:
      program: gh
      args: ["pr", "checks", "--watch", "--fail-fast"]
      cwd_mode: task_root
      timeout_secs: 600

  pr-review:
    mode: agent
    agent: reviewer
    directive: "Review the PR and merge if approved."

workflows:
  - id: scaffold
    phases:
      - implementation
      - install-deps
      - build-check
      - lint-check
      - push-branch
      - create-pr
      - wait-for-ci
      - pr-review:
          on_verdict:
            rework:
              target: implementation
```

## Critical: cwd_mode for Command Phases

**This is the #1 gotcha.** Command phases default to `cwd_mode: project_root`, which runs the command in the main repo — NOT in the task's worktree. Every command that needs to run in the worktree (push, PR create, build, install) MUST have:

```yaml
command:
  cwd_mode: task_root
```

Without this, `git push` pushes from `main` (nothing to push), `gh pr create` sees no branch, and `pnpm build` may use stale code.

### cwd_mode options:
- `project_root` — main repo directory (default)
- `task_root` — the task's git worktree (what you almost always want)
- `path` — custom relative path (requires `cwd_path`)

## Why Command Phases Over Agent Phases for Git/PR

Agent phases (mode: agent) spawn a full Claude/Codex session. For deterministic operations like `git push` and `gh pr create`, this is:
- Slow (spawns entire LLM session for a shell command)
- Unreliable (the agent might do unexpected things)
- Expensive (uses model tokens for no reason)

**Rule: Use agent phases for decisions, command phases for execution.**

| Operation | Phase Mode | Why |
|-----------|-----------|-----|
| Write code | agent | Needs intelligence |
| Run tests | command | Pass/fail, no interpretation |
| Push branch | command | Deterministic |
| Create PR | command | Deterministic |
| Wait for CI | command | Just polling |
| Review PR | agent | Needs judgment |
| Resolve conflicts | agent | Needs intelligence |

## install-deps Phase — Why It's Required

Worktrees don't have `node_modules`. The implementation agent uses Claude Code's file tools (Read/Write/Edit) which don't need deps installed. But command phases that run `pnpm build`, `pnpm test`, or `pnpm lint` will fail with "command not found" errors.

**Always add `install-deps` before any command phase that needs node_modules.**

## QA Gates Without Rework Loops

Don't add `on_verdict: rework` to CI/CD gates — it creates infinite loops:

```yaml
# BAD — infinite loop if build always fails
- build-check:
    on_verdict:
      rework:
        target: implementation

# GOOD — fail cleanly, let PO/auditor create a fix task
- build-check
- lint-check
- wait-for-ci
```

Only `pr-review` should have a rework loop (reviewer feedback is actionable).

## Conflict Resolution Pipeline

When parallel agents merge PRs, later PRs conflict. Add a rebase workflow:

```yaml
phases:
  rebase-on-main:
    mode: agent
    agent: implementer
    directive: |
      Rebase this branch onto latest main. Resolve conflicts:
      - For lockfiles: accept theirs, run pnpm install
      - For code: keep both sides, combine logic
      - Run pnpm build to verify result
    capabilities:
      mutates_state: true

  force-push:
    mode: command
    command:
      program: git
      args: ["push", "--force-with-lease", "origin", "HEAD"]
      cwd_mode: task_root

workflows:
  - id: rebase-and-retry
    phases:
      - rebase-on-main
      - force-push
      - pr-review

  - id: rework
    phases:
      - address-review
      - force-push
      - pr-review
```

The reviewer agent queues `rebase-and-retry` when it sees a conflicting PR, and `rework` when it requests changes.

## Reviewer CI Check Pattern

The reviewer MUST check CI before merging:

```yaml
pr-review:
  mode: agent
  agent: reviewer
  directive: |
    Before merging, run: gh pr checks <number>
    - If checks fail and caused by THIS PR: queue rework
    - If checks fail but pre-existing: merge anyway, create fix task
    - If checks pending: skip, next cycle will pick it up
    - If all pass: review diff and merge if approved
```

## Stale PR Detection

PRs can become stale when tasks are marked done prematurely:

```yaml
pr-review-sweep:
  directive: |
    For each open PR:
    1. Get task ID from branch name
    2. If task is "done" but NO merged PR exists:
       The task was marked done prematurely.
       Queue rebase-and-retry (don't close the PR!)
    3. If task is "done" AND a merged PR exists:
       Close with comment linking the merged PR.
```
