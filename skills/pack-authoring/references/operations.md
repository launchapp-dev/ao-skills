# Operations

Use this reference when installing, inspecting, pinning, or operating packs.

## Cron schedules

```yaml
schedules:
  - id: my-org.my-pack/nightly-scan
    cron: "0 2 * * *"
    workflow_ref: my-org.my-pack/standard
    enabled: true
```

## Resolution order

1. Project overrides in `.ao/plugins/<pack-id>/`
2. Installed packs in `~/.ao/packs/<pack-id>/<version>/`
3. Bundled packs in the AO binary

## CLI commands

### Install a pack

```bash
ao pack install --path ./my-pack
ao pack install --name my-pack --registry my-registry
ao pack install --path ./my-pack --force --activate
```

### List and inspect packs

```bash
ao pack list
ao pack list --active-only
ao pack list --source installed
ao pack inspect --pack-id my-org.my-pack
ao pack inspect --path ./my-pack
```

### Pin or disable a pack

```bash
ao pack pin --pack-id my-org.my-pack --version "=0.2.0"
ao pack pin --pack-id ao.task --source installed
ao pack pin --pack-id my-org.my-pack --disable
```

Pack selections are stored in `.ao/state/pack-selection.v1.json`.

### Marketplace

```bash
ao pack registry add --id community --url https://github.com/ao-packs/registry
ao pack registry sync --id community
ao pack search --query "review" --registry community
ao pack registry list
ao pack registry remove --id community
```

## Bundled packs

| Pack | Exports | Purpose |
|------|---------|---------|
| `ao.task` | standard, ui-ux, quick-fix, gated, triage, refine | Task workflow pipelines |
| `ao.review` | cycle | Reusable code-review and testing loop |
| `ao.requirement` | draft, refine, plan, execute | Requirement planning and execution |

## Example connector pack

```toml
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

[mcp]
servers = "mcp/servers.toml"

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
