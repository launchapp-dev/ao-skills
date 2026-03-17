# MCP Server Setup

AO exposes all its operations as an MCP (Model Context Protocol) server. This lets any MCP-aware AI assistant (Claude Code, etc.) use AO tools directly.

## Quick Setup

### Claude Code

Create `.mcp.json` in your project root:

```json
{
  "mcpServers": {
    "ao": {
      "command": "/path/to/ao",
      "args": ["--project-root", "/path/to/your/project", "mcp", "serve"]
    }
  }
}
```

Replace paths with your actual locations:
- `command`: path to the `ao` binary (e.g., `~/ao-cli/target/debug/ao` or wherever you built it)
- `--project-root`: path to the project AO manages

### Example for a project at ~/my-project

```json
{
  "mcpServers": {
    "ao": {
      "command": "/Users/you/ao-cli/target/debug/ao",
      "args": ["--project-root", "/Users/you/my-project", "mcp", "serve"]
    }
  }
}
```

### Using a Global Install

If `ao` is on your PATH:

```json
{
  "mcpServers": {
    "ao": {
      "command": "ao",
      "args": ["--project-root", ".", "mcp", "serve"]
    }
  }
}
```

## Verifying the Connection

After creating `.mcp.json`, restart your AI assistant (e.g., restart Claude Code). Then test:

1. Ask the assistant to call `ao.daemon.status` — it should return running/stopped
2. Ask it to call `ao.task.stats` — it should return task counts
3. If tools aren't available, check that the `ao` binary path is correct

## Available Tool Groups

Once connected, the assistant gets access to:

| Prefix | Tools | Purpose |
|--------|-------|---------|
| `ao.task.*` | 14 tools | Task CRUD, status, checklists, bulk ops |
| `ao.queue.*` | 7 tools | Dispatch queue management |
| `ao.daemon.*` | 11 tools | Daemon lifecycle and monitoring |
| `ao.workflow.*` | 10+ tools | Workflow execution and inspection |
| `ao.output.*` | 5 tools | Run output, artifacts, monitoring |
| `ao.requirements.*` | 5 tools | Requirement management |
| `ao.runner.*` | 3 tools | Runner health and diagnostics |
| `ao.agent.*` | 3 tools | Agent control and status |

## Claude Code Settings

To auto-approve AO MCP tools (avoid permission prompts), add to `.claude/settings.local.json`:

```json
{
  "permissions": {
    "allow": [
      "mcp__ao__ao_daemon_health",
      "mcp__ao__ao_daemon_status",
      "mcp__ao__ao_daemon_start",
      "mcp__ao__ao_daemon_stop",
      "mcp__ao__ao_daemon_events",
      "mcp__ao__ao_daemon_logs",
      "mcp__ao__ao_daemon_agents",
      "mcp__ao__ao_daemon_config",
      "mcp__ao__ao_task_list",
      "mcp__ao__ao_task_get",
      "mcp__ao__ao_task_create",
      "mcp__ao__ao_task_status",
      "mcp__ao__ao_task_update",
      "mcp__ao__ao_task_stats",
      "mcp__ao__ao_task_prioritized",
      "mcp__ao__ao_task_next",
      "mcp__ao__ao_queue_list",
      "mcp__ao__ao_queue_enqueue",
      "mcp__ao__ao_queue_drop",
      "mcp__ao__ao_workflow_list",
      "mcp__ao__ao_workflow_run",
      "mcp__ao__ao_workflow_get",
      "mcp__ao__ao_output_tail",
      "mcp__ao__ao_runner_health"
    ]
  },
  "enableAllProjectMcpServers": true
}
```

## Multiple Projects

You can manage multiple projects by running separate MCP servers:

```json
{
  "mcpServers": {
    "ao-frontend": {
      "command": "ao",
      "args": ["--project-root", "/path/to/frontend", "mcp", "serve"]
    },
    "ao-backend": {
      "command": "ao",
      "args": ["--project-root", "/path/to/backend", "mcp", "serve"]
    }
  }
}
```

Tool names will be prefixed: `mcp__ao-frontend__ao_task_list`, `mcp__ao-backend__ao_task_list`.

## Other AI Tools

Any MCP-compatible tool can connect to AO. The server uses stdio transport:

```bash
# Manual test — sends JSON-RPC over stdin/stdout
ao --project-root /path/to/project mcp serve
```

The server speaks the MCP protocol (JSON-RPC 2.0 over stdio).

## Troubleshooting

### "MCP server not found"
- Check that the `ao` binary path in `.mcp.json` is absolute and correct
- Verify `ao mcp serve` runs without errors: `ao --project-root . mcp serve`

### Tools not appearing
- Restart your AI assistant after creating/modifying `.mcp.json`
- Check `enableAllProjectMcpServers: true` in Claude settings

### "project_root" errors
- Ensure `--project-root` points to a directory with `.ao/` or a git repo
- Use absolute paths, not relative
