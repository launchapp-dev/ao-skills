---
name: skill-authoring
description: Build AO skills — YAML skill definitions, prompts, tool policies, capabilities, adapters, registries
user_invocable: true
auto_invoke: true
---

# Skill Authoring

Skills configure how AO agents behave — prompts, tool policies, model preferences, and capabilities. They're the lightweight complement to packs: while packs define *what work happens*, skills define *how agents approach that work*.

## What Skills Do

- Override or extend agent system prompts
- Restrict or grant tool access via policies
- Set model preferences per tool (Claude vs Codex vs Gemini)
- Activate specific MCP servers
- Set capability flags (writes_files, is_review, etc.)
- Provide tool-specific adapters (different behavior per CLI tool)

## Skill Definition Format

Skills are defined in YAML files with schema `ao.skills.v1`:

```yaml
skills:
  code-review:
    name: code-review
    version: "1.0.0"
    description: Professional code review with security and performance focus.
    category: review

    activation:
      tools: [claude]         # empty = all tools
      models: []              # empty = all models

    prompt:
      system: |
        You are an expert code reviewer. Focus on:
        - Security vulnerabilities (OWASP top 10)
        - Performance bottlenecks
        - Error handling gaps
        - Type safety issues
      prefix: "Review the following changes carefully."
      suffix: "Provide actionable feedback with line references."
      directives:
        - Never approve code with hardcoded secrets
        - Flag any SQL string concatenation

    tool_policy:
      allow:
        - "Read"
        - "Grep"
        - "Glob"
        - "task.*"
      deny:
        - "Write"
        - "Edit"
        - "Bash"

    model:
      preferred: claude-sonnet-4-6
      fallback:
        - claude-opus-4-6

    mcp_servers:
      - ao
      - sequential-thinking

    timeout_secs: 300

    capabilities:
      is_review: true
      writes_files: false
      mutates_state: false

    extra_args: []
    env: {}
    tags: ["review", "security", "quality"]
```

## Skill Definition Fields

| Field | Type | Required | Description |
|-------|------|:--------:|-------------|
| `name` | string | Yes | Unique skill identifier |
| `version` | string | No | Semver version |
| `description` | string | Yes | What this skill does |
| `category` | string | No | One of: `implementation`, `testing`, `review`, `research`, `documentation`, `operations`, `planning` |
| `activation` | object | No | Which tools/models trigger this skill |
| `prompt` | object | No | System prompt, prefix, suffix, directives |
| `tool_policy` | object | No | Allow/deny glob patterns for tools |
| `model` | object | No | Preferred and fallback model IDs |
| `mcp_servers` | list | No | MCP servers to activate |
| `timeout_secs` | int | No | Agent timeout override |
| `capabilities` | object | No | Boolean capability flags |
| `extra_args` | list | No | Extra CLI arguments |
| `env` | object | No | Environment variables |
| `codex_config_overrides` | list | No | Codex-specific overrides |
| `adapters` | object | No | Per-tool behavior overrides |
| `tags` | list | No | Searchable tags |

## Prompt Structure

Skills can inject prompts at multiple points:

```yaml
prompt:
  system: |
    Full system prompt override. Replaces the default.
  prefix: |
    Prepended before the phase directive.
  suffix: |
    Appended after the phase directive.
  directives:
    - Individual rules merged into the prompt.
    - Each directive is a standalone instruction.
```

Use `directives` for composable rules that combine well with other skills. Use `system` for a complete prompt override.

## Tool Policies

Control which MCP tools and CLI tools an agent can use:

```yaml
tool_policy:
  allow:
    - "task.*"           # all task tools
    - "Read"             # specific tool
    - "requirements.*"   # all requirement tools
  deny:
    - "task.delete"      # block dangerous operations
    - "Bash"             # prevent shell access
    - "git.*"            # no direct git operations
```

Patterns use glob matching. `deny` takes precedence over `allow`.

## Capabilities

Boolean flags that control agent behavior at the system level:

```yaml
capabilities:
  writes_files: true           # can write to filesystem
  mutates_state: true          # can change AO state (tasks, queue, etc.)
  requires_commit: true        # must commit changes
  enforce_product_changes: true # enforce product-level changes
  is_research: true            # research-oriented (no code changes expected)
  is_ui_ux: true               # UI/UX focused
  is_review: true              # review/analysis phase
  is_testing: true             # testing phase
  is_requirements: true        # requirements-focused
```

Aliases are supported: `write_files`, `file_write`, `can_write` all map to `writes_files`.

## Tool-Specific Adapters

Override skill behavior per CLI tool:

```yaml
adapters:
  claude:
    model: claude-opus-4-6
    extra_args: ["--no-stream"]
    mcp_servers: ["ao", "sequential-thinking"]
    prompt_override:
      suffix: "Use extended thinking for complex reviews."

  codex:
    model: o3
    tool_policy:
      allow: ["*"]
    extra_args: ["--full-auto"]

  gemini:
    model: gemini-3.1-pro-preview
    prompt_override:
      system: "You are a code reviewer using Gemini."
```

Adapters let a single skill work across different AI tools with optimized settings for each.

## Activation Filters

Control when a skill activates:

```yaml
activation:
  tools: [claude, codex]    # only activate for these CLI tools
  models: []                # empty = all models
```

Leave both empty for universal activation.

## Skill Sources and Priority

Skills are loaded from multiple sources. Last match wins:

| Priority | Source | Location |
|:--------:|--------|----------|
| 1 (lowest) | Builtin | Embedded in AO binary |
| 2 | Installed | `~/.ao/state/skills-registry.v1.json` |
| 3 | User | `~/.ao/config/skill_definitions/*.yaml` |
| 4 (highest) | Project | `.ao/skill_definitions/*.yaml` |

Project skills override everything else, letting you customize behavior per-repo.

## File Locations

### User-Level Skills (apply to all projects)

Place YAML files in `~/.ao/config/skill_definitions/`:

```
~/.ao/config/skill_definitions/
├── my-review-skill.yaml
├── my-testing-skill.yaml
└── my-planning-skill.yaml
```

### Project-Level Skills (apply to one project)

Place YAML files in `.ao/skill_definitions/`:

```
.ao/skill_definitions/
├── project-review.yaml
└── project-testing.yaml
```

### Multiple Skills Per File

A single YAML file can define multiple skills:

```yaml
skills:
  security-review:
    name: security-review
    description: Security-focused code review.
    category: review
    prompt:
      directives:
        - Check for SQL injection
        - Check for XSS vulnerabilities
    capabilities:
      is_review: true

  performance-review:
    name: performance-review
    description: Performance-focused code review.
    category: review
    prompt:
      directives:
        - Check for N+1 queries
        - Flag unnecessary re-renders
    capabilities:
      is_review: true
```

## CLI Commands

### Search Skills

```bash
ao skill search --query "review"
ao skill search --source user
ao skill search --registry community
```

### Install a Skill

```bash
ao skill install --name code-review --registry community
ao skill install --name code-review --version "^1.0" --allow-prerelease
```

### List Skills

```bash
ao skill list                   # all skills
ao skill list --source project  # project-level only
```

### Show Skill Details

```bash
ao skill show --name code-review
```

### Update a Skill

```bash
ao skill update --name code-review --version "^2.0"
```

### Publish a Skill

```bash
ao skill publish --name code-review --version "1.0.0" --source my-org --registry community
```

### Skill Registries

```bash
ao skill registry add --id community --url https://github.com/ao-skills/registry
ao skill registry list
ao skill registry remove --id community
```

## Using Skills in Workflows

### In Agent Profiles

Reference skills in agent definitions to compose behaviors:

```yaml
agents:
  my-reviewer:
    skills:
      - code-review
      - security-review
    tool: claude
    model: claude-sonnet-4-6
```

### In Phase Definitions

Activate additional skills for specific phases:

```yaml
phases:
  security-audit:
    mode: agent
    agent_id: my-reviewer
    directive: "Audit the codebase for security issues."
    skills:
      - security-review
```

## Example: Complete Implementation Skill

```yaml
skills:
  careful-implementer:
    name: careful-implementer
    version: "1.0.0"
    description: Implementation skill that prioritizes correctness over speed.
    category: implementation

    prompt:
      system: |
        You are a careful software engineer. Before writing code:
        1. Read existing tests to understand expected behavior
        2. Check for similar patterns in the codebase
        3. Write code that follows existing conventions
        4. Run tests before committing
        5. Commit with descriptive messages
      directives:
        - Never modify test files unless the task explicitly requires it
        - Always check for existing utility functions before writing new ones
        - Use the project's existing error handling patterns

    tool_policy:
      allow:
        - "Read"
        - "Write"
        - "Edit"
        - "Glob"
        - "Grep"
        - "Bash"
        - "task.*"
        - "output.*"
      deny:
        - "task.delete"
        - "requirements.delete"

    model:
      preferred: claude-sonnet-4-6
      fallback:
        - claude-opus-4-6

    mcp_servers:
      - ao
      - context7

    capabilities:
      writes_files: true
      mutates_state: true
      requires_commit: true
      is_review: false
      is_testing: false

    adapters:
      gemini:
        model: gemini-3.1-pro-preview
        prompt_override:
          suffix: "Use your 1M context window to read broadly before making changes."

    tags: ["implementation", "careful", "quality"]
```

## Design Principles

1. **Keep skills focused.** One skill = one behavior. Compose via lists in agent profiles.
2. **Use directives over system prompts** when possible — they compose better with other skills.
3. **Scope tool policies tightly.** A review skill should deny Write/Edit. An implementation skill should deny task.delete.
4. **Use adapters** when different AI tools need different settings for the same skill.
5. **Put project-specific skills in `.ao/skill_definitions/`** — they override everything and stay with the repo.
6. **Put personal preferences in `~/.ao/config/skill_definitions/`** — they apply to all your projects.
7. **Tag skills** for discoverability in registries and search.
