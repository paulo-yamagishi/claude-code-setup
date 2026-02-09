# Permission Rules Deep Dive

This is the detailed reference for Claude Code permission rules. For the settings hierarchy and basic schema, see [settings-reference.md](settings-reference.md).

---

## Permission Rule Format

Rules follow the pattern `Tool` or `Tool(specifier)`:

- `"Bash(npm test)"` -- Allow only this exact command
- `"Bash(npm *)"` -- Allow any command starting with `npm`
- `"Bash(*)"` -- Allow all bash commands (use with caution)
- `"mcp__server-name__tool-name"` -- Allow a specific MCP tool
- `"mcp__server-name__*"` -- Allow all tools from an MCP server
- `"Task(AgentName)"` -- Control access to a specific subagent

## Read/Edit Permission Rules

Read and Edit permissions support gitignore-style path patterns:

```json
{
  "permissions": {
    "allow": [
      "Read(src/**)",
      "Edit(src/**)"
    ],
    "deny": [
      "Read(.env*)",
      "Edit(config/secrets.json)"
    ]
  }
}
```

Path pattern resolution:

| Pattern | Resolves relative to |
|---|---|
| `//path` | Absolute filesystem path |
| `~/path` | User home directory |
| `/path` | Settings file location |
| `path` | Current working directory |

## defaultMode Values

- `default` -- Ask the user before running tools (default)
- `acceptEdits` -- Auto-approve file edits, ask for others
- `bypassPermissions` -- Allow everything without asking
- `plan` -- Require plan approval before implementation
- `delegate` -- Delegate permission decisions to subagents
- `dontAsk` -- Deny rather than prompting

## Evaluation Order

Permission rules are evaluated in this order: **deny → ask → allow** (first match wins).

```json
{
  "permissions": {
    "allow": [
      "Bash(npm test)",
      "Bash(npm run lint)",
      "Bash(npm run build)",
      "Read",
      "Write",
      "Edit",
      "Glob",
      "Grep"
    ],
    "deny": [
      "Bash(rm -rf *)"
    ],
    "ask": [
      "Bash(git push *)"
    ],
    "defaultMode": "default"
  }
}
```

## Managed-Only Permission Settings

These fields are only available in managed settings (set by enterprise administrators):

| Field | Description |
|---|---|
| `disableBypassPermissionsMode` | Prevent users from using bypass permissions mode |
| `allowManagedPermissionRulesOnly` | Only managed permission rules apply; user/project rules ignored |
| `allowManagedHooksOnly` | Only managed hooks can run; user/project hooks ignored |
| `strictKnownMarketplaces` | Restrict plugin installs to approved marketplaces only |
