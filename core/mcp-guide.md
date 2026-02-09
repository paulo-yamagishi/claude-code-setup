# MCP: Model Context Protocol Setup and Best Practices

MCP (Model Context Protocol) lets Claude use external tools and access external data sources. An MCP server exposes tools that Claude can call, extending its capabilities beyond built-in tools.

---

## Transports

MCP servers communicate with Claude through one of three transport mechanisms:

### HTTP (recommended)

The recommended transport for remote MCP servers. Uses standard HTTP for communication.

```json
{
  "type": "http",
  "url": "https://mcp.example.com/api"
}
```

### SSE (deprecated)

Server-Sent Events transport. Used for shared team servers or cloud-hosted tools. Deprecated in favor of HTTP.

```json
{
  "type": "sse",
  "url": "https://mcp.example.com/sse"
}
```

### stdio (local)

The server runs as a local child process. Communication happens over standard input/output. Best for local tools.

```json
{
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/dir"]
}
```

---

## CLI Commands

Manage MCP servers with the `claude mcp` subcommands:

| Command | Description |
|---|---|
| `claude mcp add` | Add an MCP server |
| `claude mcp add-json` | Add server from JSON config |
| `claude mcp add-from-claude-desktop` | Import servers from Claude Desktop |
| `claude mcp list` | List configured servers |
| `claude mcp get` | Get details for a server |
| `claude mcp remove` | Remove a server |
| `claude mcp reset-project-choices` | Reset per-project approval choices |
| `claude mcp serve` | Start Claude Code as an MCP server |

---

## Configuration Scopes

MCP servers can be configured at three levels:

| Scope | File | Shared with Team | Use Case |
|---|---|---|---|
| Project | `.mcp.json` | Yes (version controlled) | Team-shared servers (GitHub, DB) |
| Local | `~/.claude.json` (under project path) | No | Personal dev servers (default scope) |
| User | `~/.claude.json` | No | Global personal servers |

---

## Configuration Format

### .mcp.json (project scope)

```json
{
  "mcpServers": {
    "server-name": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "package-name"],
      "env": {
        "API_KEY": "${API_KEY}"
      }
    }
  }
}
```

Environment variable expansion is supported in `.mcp.json` using `${VAR}` and `${VAR:-default}` syntax.

### Fields

- **type** -- Transport: `stdio`, `sse`, or `http`
- **command** -- Executable to run (stdio only)
- **args** -- Arguments to pass to the command (stdio only)
- **url** -- Server URL (sse and http only)
- **env** -- Environment variables for the server process

---

## OAuth Support

Use the `/mcp` command to initiate a browser-based OAuth flow for servers that require authentication.

## Tool Search

When configured tools exceed 10% of the context window, tool search is auto-enabled. This helps Claude efficiently find the right tool without overloading context.

## MCP Resources

Access MCP server resources using the `@` syntax:

```
@server:protocol://resource/path
```

## MCP Prompts

Invoke MCP server prompts using the `/` syntax:

```
/mcp__servername__promptname
```

---

## Popular MCP Servers

### Filesystem

Access files outside the sandbox or with elevated permissions:

```json
{
  "mcpServers": {
    "filesystem": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/me/documents"]
    }
  }
}
```

### GitHub

Manage pull requests, issues, and repositories:

```json
{
  "mcpServers": {
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

### PostgreSQL

Query and manage PostgreSQL databases:

```json
{
  "mcpServers": {
    "postgres": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres", "postgresql://localhost:5432/mydb"]
    }
  }
}
```

---

## Best Practices

### Limit the number of servers

Each MCP server adds tool descriptions to Claude's context. Too many servers waste context and can confuse tool selection. Enable only the servers you actively need.

### Use project scope for team servers

If the whole team uses the same GitHub or database server, configure it in `.mcp.json` so everyone gets it automatically.

### Use local scope for personal servers

Dev-specific servers (local databases, personal tools) are added with the default local scope to avoid cluttering team config.

### Never commit secrets

Use environment variable references (`${VAR}` or `${VAR:-default}`) for API keys and tokens. Set the actual values in your shell environment:

```bash
# In your .bashrc or .zshrc
export GITHUB_TOKEN="ghp_..."
export LINEAR_API_KEY="lin_..."
```

### Test connections

Use the `/mcp` command inside Claude Code to verify MCP servers are connected and list available tools.

---

## Troubleshooting

### Server fails to start

- Verify the command exists: run it manually in your terminal
- Check that `npx` can download the package (network/proxy issues)
- Look at Claude Code logs for error messages

### Tools not appearing

- Run `/mcp` to check server status
- Verify the server is configured in the correct scope
- Check that the server process is running (for stdio, check if the process started)

### Authentication errors

- Verify environment variables are set in your shell
- Check that `${VAR}` references in settings match your actual env var names
- For GitHub, ensure your token has the required scopes

### Timeout issues

- stdio servers must start quickly; slow startup can cause timeouts
- For remote servers (sse/http), check network connectivity
- Consider running resource-heavy servers locally instead of remotely
