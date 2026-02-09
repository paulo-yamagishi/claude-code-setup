# MCP Server Templates for /bootstrap

Include only the servers the user selected. Wrap in `{ "mcpServers": { ... } }` and write to `.mcp.json`.

## GitHub

```json
{
  "github": {
    "type": "stdio",
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-github"],
    "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}" }
  }
}
```

## Filesystem

```json
{
  "filesystem": {
    "type": "stdio",
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/dir"]
  }
}
```

## PostgreSQL

```json
{
  "postgres": {
    "type": "stdio",
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-postgres"],
    "env": { "DATABASE_URL": "${DATABASE_URL}" }
  }
}
```

## Slack

```json
{
  "slack": {
    "type": "stdio",
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-slack"],
    "env": { "SLACK_BOT_TOKEN": "${SLACK_BOT_TOKEN}", "SLACK_TEAM_ID": "${SLACK_TEAM_ID}" }
  }
}
```

## Environment Variables

Create `.claude/ENV.md` with required variables for each configured MCP server:

```markdown
# Environment Variables for MCP Servers

Add these to your shell profile (`~/.zshrc` or `~/.bashrc`):

[For each configured MCP:]
export GITHUB_TOKEN="your_token_here"  # Get from: https://github.com/settings/tokens
export DATABASE_URL="postgresql://..."  # Your database connection string
[etc.]

Restart your terminal after adding these.
```
