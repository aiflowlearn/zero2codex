# MCP (Model Context Protocol) Configuration

## Overview

Codex supports MCP servers to connect external tools and data sources. MCP extends Codex's capabilities by providing access to databases, APIs, and other services through a standardized protocol.

## MCP Server Types

### STDIO Servers

Run a local process and communicate via stdin/stdout:

```toml
# ~/.codex/config.toml
[mcp_servers.my-stdio]
command = "node"
args = ["server.js"]
env = { API_KEY = "secret" }
env_vars = ["EXISTING_VAR"]  # pass through from host
cwd = "/path/to/workdir"
```

### Streamable HTTP Servers

Connect to remote MCP endpoints:

```toml
[mcp_servers.my-http]
url = "https://mcp.example.com/sse"
bearer_token_env_var = "MCP_BEARER_TOKEN"
http_headers = { X-Custom = "value" }
env_http_headers = { X-API-KEY = "API_KEY_ENV_VAR" }
```

## Common Options

| Option | Description |
|--------|-------------|
| `startup_timeout_sec` | Seconds to wait for server startup |
| `tool_timeout_sec` | Seconds per tool call |
| `enabled` | Enable/disable server (default true) |
| `required` | Fail if server can't start (default false) |
| `enabled_tools` | Whitelist of tools from this server |
| `disabled_tools` | Blacklist of tools from this server |

## CLI Commands

Add MCP servers from the command line:

```bash
# Add STDIO server
codex mcp add my-server --env API_KEY=xxx -- node server.js

# Add HTTP server
codex mcp add my-http --url https://mcp.example.com/sse
```

## TUI Slash Command

Type `/mcp` in the CLI to list configured MCP tools and their status.

## Example: GitHub MCP Server

```toml
[mcp_servers.github]
command = "npx"
args = ["-y", "@modelcontextprotocol/server-github"]
env = { GITHUB_PERSONAL_ACCESS_TOKEN = "ghp_..." }
```

After adding, use `/mcp` to confirm the server is connected, then ask Codex to use GitHub tools:

```
List open pull requests in the current repository
```

## Troubleshooting

**Server won't start:**
- Check the command path and arguments
- Verify environment variables are set correctly
- Check `startup_timeout_sec` — increase if the server is slow to initialize

**Tools not appearing:**
- Run `/mcp` to see if the server is connected
- Check `enabled_tools` / `disabled_tools` configuration
- Verify the server implements the MCP protocol correctly
