---
name: pack-authoring
description: Build workflow packs — pack.toml manifest, directory structure, agents, phases, MCP servers, marketplace publishing
user_invocable: true
auto_invoke: true
---

# Pack Authoring

Workflow packs are the primary extension mechanism for AO. A pack bundles workflow definitions, agent overlays, MCP server descriptors, and cron schedules into an installable, versioned package.

## Pack Kinds

| Kind | Purpose | Example |
|------|---------|---------|
| `domain-pack` | End-to-end workflow for a subject type (task, requirement) | ao.task, ao.review |
| `connector-pack` | Integrates an external service via MCP | Jira connector, Linear connector |
| `capability-pack` | Adds reusable phases or agent behaviors | QA gates, security scanning |

## Directory Structure

```
my-pack/
├── pack.toml                          # Pack manifest (required)
├── workflows/
│   └── my-workflows.yaml              # Workflow and phase definitions
├── runtime/
│   ├── agent-runtime.overlay.yaml     # Agent profiles and phase overrides
│   └── workflow-runtime.overlay.yaml  # Workflow-level overlays (optional)
├── mcp/
│   └── servers.toml                   # MCP server descriptors (optional)
├── schedules/
│   └── schedules.yaml                 # Cron schedules (optional)
└── subjects/
    └── schemas.yaml                   # Subject type schemas (optional)
```

## Pack Manifest (`pack.toml`)

Every pack requires a `pack.toml` at its root:

```toml
schema = "ao.pack.v1"
id = "my-org.my-pack"
version = "0.1.0"
kind = "domain-pack"
title = "My Pack"
description = "What this pack does."

[ownership]
mode = "project"         # bundled | installed | project

[compatibility]
ao_core = ">=0.1.0"
workflow_schema = "v2"
subject_schema = "v2"

[subjects]
kinds = ["ao.task"]      # subject types this pack handles
default_kind = "ao.task"

[workflows]
root = "workflows"       # directory containing workflow YAML
exports = [              # workflow refs this pack exposes
  "my-org.my-pack/standard",
  "my-org.my-pack/quick-fix",
]

[runtime]
agent_overlay = "runtime/agent-runtime.overlay.yaml"
# workflow_overlay = "runtime/workflow-runtime.overlay.yaml"  # optional

[[dependencies]]
id = "ao.review"
version = ">=0.1.0"
reason = "Uses the review cycle for PR review."

# optional = true        # won't block activation if missing

[permissions]
tools = ["ao", "gh", "pnpm"]          # MCP tools and CLI programs used
mcp_namespaces = []                    # MCP namespaces accessed

[secrets]
required = []                          # env vars that must be set
optional = ["GITHUB_TOKEN"]            # env vars that enhance behavior

# For connector packs that need Node/Python:
# [[runtime.requirements]]
# runtime = "node"       # node | python | uv | npm | pnpm
# version = ">=18.0.0"
# reason = "MCP server requires Node 18+"
```

### Manifest Fields Reference

| Section | Field | Required | Description |
|---------|-------|:--------:|-------------|
| top-level | `schema` | Yes | Always `"ao.pack.v1"` |
| top-level | `id` | Yes | Unique pack ID (e.g., `"my-org.my-pack"`) |
| top-level | `version` | Yes | Semver version |
| top-level | `kind` | Yes | `domain-pack`, `connector-pack`, or `capability-pack` |
| top-level | `title` | Yes | Human-readable name |
| top-level | `description` | Yes | What the pack does |
| `ownership` | `mode` | Yes | `bundled`, `installed`, or `project` |
| `compatibility` | `ao_core` | Yes | Minimum AO version (semver range) |
| `compatibility` | `workflow_schema` | Yes | Workflow schema version (`v2`) |
| `compatibility` | `subject_schema` | Yes | Subject schema version (`v2`) |
| `subjects` | `kinds` | No | Subject types this pack handles |
| `subjects` | `default_kind` | No | Default subject type |
| `workflows` | `root` | Yes | Directory containing workflow YAML |
| `workflows` | `exports` | Yes | List of workflow refs to expose |
| `runtime` | `agent_overlay` | No | Path to agent runtime overlay YAML |
| `runtime` | `workflow_overlay` | No | Path to workflow runtime overlay YAML |
| `dependencies` | `id`, `version` | No | Other packs this depends on |
| `permissions` | `tools` | No | CLI tools and MCP tools used |
| `permissions` | `mcp_namespaces` | No | MCP namespaces accessed |
| `secrets` | `required` | No | Env vars that must be set |
| `secrets` | `optional` | No | Env vars that enhance behavior |
| `native_module` | `feature`, `module_id` | No | Rust feature gate (advanced) |

## Workflow Definitions

Place workflow YAML in the `workflows/` directory. The format is the same as `custom.yaml` but with pack-qualified IDs.

```yaml
phase_catalog:
  my-analysis:
    label: Analysis
    description: Analyze the codebase for issues.
    category: review
    tags: ["analysis", "review"]

workflows:
  - id: my-org.my-pack/standard
    name: My Standard Workflow
    description: Full analysis and fix pipeline.
    phases:
      - my-analysis
      - implementation
      - workflow_ref: ao.review/cycle    # reference another pack's workflow

  - id: my-org.my-pack/quick-fix
    name: Quick Fix
    description: Implement and test only.
    phases:
      - implementation
      - testing
```

### Phase Catalog

The `phase_catalog` section documents phases with metadata. This is optional but recommended for discoverability:

```yaml
phase_catalog:
  my-phase:
    label: Human-Readable Name
    description: What this phase does.
    category: planning | review | verification | implementation
    tags: ["searchable", "tags"]
```

### Sub-Workflow References

Phases can reference workflows from other packs:

```yaml
phases:
  - workflow_ref: ao.review/cycle       # runs the entire review cycle inline
```

### Rework Loops

Only use rework on agent phases (reviews), never on command phases:

```yaml
phases:
  - implementation
  - code-review:
      on_verdict:
        rework:
          target: implementation
      max_rework_attempts: 3
```

## Agent Runtime Overlay

The `runtime/agent-runtime.overlay.yaml` defines agent profiles and phase execution for the pack.

```yaml
tools_allowlist:
  - ao
  - gh
  - pnpm

agents:
  my-analyzer:
    description: Analyzes the codebase for issues.
    system_prompt: |
      You are a code analyzer. Inspect the codebase and report issues.
      Use AO MCP to create tasks for any problems found.
    role: reviewer
    mcp_servers: ["ao"]
    tool_policy:
      allow:
        - task.*
        - output.*
      deny:
        - task.delete
        - project.remove
    skills:
      - code-review
    capabilities:
      is_review: true
      implementation: false
    tool: claude
    model: claude-sonnet-4-6

phases:
  my-analysis:
    mode: agent
    agent_id: my-analyzer
    directive: Analyze the codebase and report findings via AO MCP.
    capabilities:
      mutates_state: true
    runtime:
      tool: claude
      model: claude-sonnet-4-6
    decision_contract:
      required_evidence: []
      min_confidence: 0.7
      max_risk: medium
      allow_missing_decision: false
```

### Agent Profile Fields

| Field | Type | Description |
|-------|------|-------------|
| `description` | string | What this agent does |
| `system_prompt` | string | Instructions for the agent |
| `role` | string | Semantic role (e.g., `reviewer`, `product_owner`, `engineering_manager`) |
| `mcp_servers` | list | MCP servers this agent can access |
| `tool_policy` | object | `allow` and `deny` glob patterns for MCP tools |
| `skills` | list | Skill identifiers to activate |
| `capabilities` | object | Boolean flags (planning, implementation, testing, code_review, etc.) |
| `tool` | string | CLI tool (claude, codex, gemini, oai-runner) |
| `model` | string | LLM model ID |
| `fallback_models` | list | Fallback models if primary fails |
| `timeout_secs` | int | Agent timeout |
| `max_attempts` | int | Retry attempts |
| `max_continuations` | int | Max continuations per phase |
| `extra_args` | list | Extra CLI arguments |

### Phase Definition Fields

| Field | Type | Description |
|-------|------|-------------|
| `mode` | string | `agent`, `command`, or `manual` |
| `agent_id` | string | Agent profile to use (for `mode: agent`) |
| `directive` | string | Instructions for the phase |
| `capabilities` | object | `mutates_state`, `writes_files`, `requires_commit`, etc. |
| `runtime` | object | Override tool, model, timeout, etc. for this phase |
| `decision_contract` | object | Required evidence, confidence, risk thresholds |
| `output_contract` | object | Expected output structure |
| `skills` | list | Additional skills to activate |
| `command` | object | Command definition (for `mode: command`) |
| `manual` | object | Manual approval config (for `mode: manual`) |

### Decision Contract

Controls how phase outcomes are evaluated:

```yaml
decision_contract:
  required_evidence:
    - code_review_clean    # or: tests_passed, requirements_met, etc.
  min_confidence: 0.7      # 0.0-1.0
  max_risk: medium          # low, medium, high
  allow_missing_decision: false
```

Evidence kinds: `tests-passed`, `tests-failed`, `code-review-clean`, `code-review-issues`, `files-modified`, `requirements-met`, `research-complete`, `manual-verification`, `custom`.

## MCP Server Descriptors

Connector packs can ship MCP server definitions in `mcp/servers.toml`:

```toml
[[server]]
id = "jira"
command = "npx"
args = ["-y", "@modelcontextprotocol/server-jira"]
required_env = ["JIRA_BASE_URL", "JIRA_API_TOKEN"]
tool_namespace = "jira"
startup = "phase-local"
```

Servers are namespaced to the pack and started on-demand when a phase references them.

## Cron Schedules

Add recurring workflows in `schedules/schedules.yaml`:

```yaml
schedules:
  - id: my-org.my-pack/nightly-scan
    cron: "0 2 * * *"
    workflow_ref: my-org.my-pack/standard
    enabled: true
```

## Pack Resolution Order

When multiple sources provide the same pack ID, AO uses first-match:

1. **Project overrides** — `.ao/plugins/<pack-id>/` (highest priority)
2. **Installed packs** — `~/.ao/packs/<pack-id>/<version>/`
3. **Bundled packs** — embedded in the AO binary (lowest priority)

This lets projects override bundled behavior without modifying AO itself.

## CLI Commands

### Install a Pack

```bash
# From a local directory
ao pack install --path ./my-pack

# From a marketplace
ao pack install --name my-pack --registry my-registry

# Force overwrite + activate immediately
ao pack install --path ./my-pack --force --activate
```

### List Packs

```bash
ao pack list                    # all packs
ao pack list --active-only      # only enabled packs
ao pack list --source installed # only installed packs
```

### Inspect a Pack

```bash
ao pack inspect --pack-id my-org.my-pack
ao pack inspect --path ./my-pack    # inspect before installing
```

### Pin / Disable a Pack

```bash
# Pin to a specific version
ao pack pin --pack-id my-org.my-pack --version "=0.2.0"

# Prefer installed over bundled
ao pack pin --pack-id ao.task --source installed

# Disable a pack for this project
ao pack pin --pack-id my-org.my-pack --disable
```

Pack selections are stored in `.ao/state/pack-selection.v1.json`.

### Marketplace

```bash
# Register a marketplace
ao pack registry add --id community --url https://github.com/ao-packs/registry

# Sync the catalog
ao pack registry sync --id community

# Search for packs
ao pack search --query "review" --registry community

# List registries
ao pack registry list

# Remove a registry
ao pack registry remove --id community
```

## Bundled Packs

AO ships with three first-party packs:

| Pack | Exports | Purpose |
|------|---------|---------|
| `ao.task` | standard, ui-ux, quick-fix, gated, triage, refine | Task workflow pipelines |
| `ao.review` | cycle | Reusable code-review + testing loop |
| `ao.requirement` | draft, refine, plan, execute | Requirement planning and materialization |

These are always available. Override them with project overrides in `.ao/plugins/`.

## Design Principles

1. **Pack-qualify all IDs.** Use `my-org.my-pack/workflow-name` to avoid collisions.
2. **Declare all permissions.** Undeclared tools cause activation errors.
3. **Declare secrets.** Required env vars are checked at activation — don't let agents fail at runtime.
4. **Use sub-workflow refs** to compose with bundled packs (e.g., `workflow_ref: ao.review/cycle`).
5. **Keep agents focused.** Each agent should have a clear role with scoped tool policies.
6. **Use decision contracts.** They prevent agents from advancing phases without proper evidence.
7. **Test with `ao pack inspect`** before installing to catch manifest errors early.

## Example: Complete Connector Pack

A pack that integrates a Jira connector:

```toml
# pack.toml
schema = "ao.pack.v1"
id = "my-org.jira-sync"
version = "0.1.0"
kind = "connector-pack"
title = "Jira Sync"
description = "Sync AO tasks with Jira issues."

[ownership]
mode = "project"

[compatibility]
ao_core = ">=0.1.0"
workflow_schema = "v2"
subject_schema = "v2"

[workflows]
root = "workflows"
exports = ["my-org.jira-sync/import", "my-org.jira-sync/export"]

[runtime]
agent_overlay = "runtime/agent-runtime.overlay.yaml"

[permissions]
tools = ["ao"]
mcp_namespaces = ["jira"]

[secrets]
required = ["JIRA_BASE_URL", "JIRA_API_TOKEN"]

[[runtime.requirements]]
runtime = "node"
version = ">=18.0.0"
reason = "Jira MCP server requires Node 18+"
```
