# AO Skills

A Claude Code plugin with skills for using [AO](https://github.com/launchapp-dev/ao-cli) — an autonomous agent orchestrator for software development workflows.

## Install

### From Marketplace
```bash
/plugin marketplace add launchapp-dev/ao-skills
```

### From Local Directory
```bash
claude --plugin-dir ~/ao-skills
```

Or clone and point to it:
```bash
git clone https://github.com/launchapp-dev/ao-skills.git ~/ao-skills
claude --plugin-dir ~/ao-skills
```

## Slash Commands

| Command | Description |
|---------|-------------|
| `/setup-ao` | Set up AO in the current project — init, MCP, first workflow |
| `/getting-started` | Install AO, core concepts, first task and workflow |
| `/mcp-setup` | Create `.mcp.json` and connect AI tools to AO |
| `/workflow-authoring` | Write custom workflow YAML — agents, phases, crons |
| `/pack-authoring` | Build workflow packs — manifest, agents, phases, marketplace |
| `/skill-authoring` | Build AO skills — prompts, tool policies, capabilities |
| `/troubleshooting` | Common AO issues and fixes |

## Auto-Invoked Reference Skills

These skills are automatically loaded by Claude when contextually relevant:

| Skill | Description |
|-------|-------------|
| configuration | Project config, daemon config, agent runtime, state layout |
| task-management | Full task lifecycle — create, list, update, block/unblock |
| daemon-operations | Start, monitor, and troubleshoot the daemon |
| queue-management | Dispatch queue operations |
| mcp-tools | Complete `ao.*` MCP tool reference |
| workflow-patterns | Battle-tested pipeline patterns from 150+ autonomous PRs |
| agent-personas | Product lifecycle agents — PO, architect, auditor, docs-writer |
| mcp-servers-for-agents | Connect agents to Context7, package-version, memory, GitHub |
| pack-authoring | Build workflow packs with pack.toml, agent overlays, MCP descriptors |
| skill-authoring | Build AO skills with YAML definitions, tool policies, adapters |

## Usage

Once installed, Claude can help you:

- **Set up AO**: `/setup-ao` walks through project init, MCP config, and first workflow
- **Write workflows**: Ask Claude to create or update `.ao/workflows.yaml` and `.ao/workflows/*.yaml`
- **Manage tasks**: Claude can create, prioritize, and enqueue tasks via MCP tools
- **Debug issues**: `/troubleshooting` covers daemon crashes, workflow failures, queue problems, and the new live log streaming flow via `ao daemon stream`
- **Configure agents**: Claude knows how to set up persona agents for the full product lifecycle
- **Build packs**: `/pack-authoring` guides you through creating installable workflow packs
- **Build skills**: `/skill-authoring` covers creating reusable agent behavior definitions
