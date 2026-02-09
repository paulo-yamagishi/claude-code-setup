# Settings Hierarchy and Configuration Reference

Claude Code uses a layered settings system. Higher-priority settings override lower ones.

---

## Settings Scopes

| Priority | Scope | Location | Committed to Git |
|---|---|---|---|
| 1 | Managed | `managed-settings.json` | N/A |
| 2 | CLI flags | Command line arguments | No |
| 3 | Local | `.claude/settings.local.json` | No (gitignored) |
| 4 | Project | `.claude/settings.json` | Yes |
| 5 | User | `~/.claude/settings.json` | No |

### Managed Settings

Set by enterprise administrators. Cannot be overridden by any other scope. Used to enforce security policies across an organization.

Managed settings file locations:

| Platform | Path |
|---|---|
| macOS | `/Library/Application Support/ClaudeCode/managed-settings.json` |
| Linux | `/etc/claude-code/managed-settings.json` |
| Windows | `C:\Program Files\ClaudeCode\managed-settings.json` |

#### Managed-Only Settings

These fields are only available in managed settings and cannot be set at other scopes:

| Field | Description |
|---|---|
| `disableBypassPermissionsMode` | Prevent users from using bypass permissions mode |
| `allowManagedPermissionRulesOnly` | Only managed permission rules apply; user/project rules ignored |
| `allowManagedHooksOnly` | Only managed hooks can run; user/project hooks ignored |
| `strictKnownMarketplaces` | Restrict plugin installs to approved marketplaces only |

### CLI Flags

Runtime overrides passed when launching Claude:

```bash
claude --model claude-opus-4-6 --max-turns 5
```

### Local Settings (.claude/settings.local.json)

Personal settings for a specific project. Gitignored by default. Use for local tool paths, personal MCP servers, or overrides you don't want to share with the team.

### Project Settings (.claude/settings.json)

Shared project settings committed to version control. The team-wide baseline configuration.

### User Settings (~/.claude/settings.json)

Personal global defaults that apply across all projects.

---

## Key Settings Fields

### permissions

Control which tools Claude can use. Permission rules use the format `Tool` or `Tool(specifier)`. Evaluation order: deny -> ask -> allow (first match wins).

```json
{
  "permissions": {
    "allow": [
      "Bash(npm test)",
      "Bash(npm run lint)",
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
    "defaultMode": "askForPermission"
  }
}
```

#### Permission Rule Format

Rules follow the pattern `Tool` or `Tool(specifier)`:

- `"Bash(npm test)"` -- Allow only this exact command
- `"Bash(npm *)"` -- Allow any command starting with `npm`
- `"Bash(*)"` -- Allow all bash commands (use with caution)
- `"mcp__server-name__tool-name"` -- Allow a specific MCP tool
- `"mcp__server-name__*"` -- Allow all tools from an MCP server
- `"Task(AgentName)"` -- Control access to a specific subagent

#### Read/Edit Permission Rules

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

#### defaultMode Values

- `askForPermission` -- Ask the user before running tools (default)
- `acceptEdits` -- Auto-approve file edits, ask for others
- `bypassPermissions` -- Allow everything without asking
- `denyEdits` -- Deny file modifications

### model

Override the default model. Accepts aliases or full model IDs:

```json
{
  "model": "claude-opus-4-6"
}
```

### effortLevel

Control reasoning effort:

```json
{
  "effortLevel": "high"
}
```

Values: `low`, `medium`, `high`.

### fastMode

Enable fast mode for quicker responses with reduced reasoning:

```json
{
  "fastMode": true
}
```

### outputStyle

Control Claude's output formatting style:

```json
{
  "outputStyle": "concise"
}
```

### env

Environment variables injected into Claude's session:

```json
{
  "env": {
    "NODE_ENV": "development",
    "DATABASE_URL": "postgresql://localhost:5432/mydb"
  }
}
```

### additionalDirectories

Extend the set of working directories Claude can access:

```json
{
  "additionalDirectories": [
    "/path/to/shared-libs",
    "../sibling-project"
  ]
}
```

### sandbox

Sandbox configuration for restricting Claude's execution environment:

```json
{
  "sandbox": {
    "mode": "automatic",
    "network": {
      "allowedDomains": ["api.example.com"],
      "httpProxyPort": 8080,
      "socksProxyPort": 1080
    },
    "excludedCommands": ["docker"],
    "allowUnsandboxedCommands": false,
    "enableWeakerNestedSandbox": false
  }
}
```

| Field | Description |
|---|---|
| `sandbox.mode` | Sandbox mode (`automatic`, `enabled`, `disabled`) |
| `sandbox.network.allowedDomains` | Domains allowed for network access |
| `sandbox.network.httpProxyPort` | HTTP proxy port for sandboxed traffic |
| `sandbox.network.socksProxyPort` | SOCKS proxy port for sandboxed traffic |
| `sandbox.excludedCommands` | Commands excluded from sandbox restrictions |
| `sandbox.allowUnsandboxedCommands` | Allow commands to run outside sandbox |
| `sandbox.enableWeakerNestedSandbox` | Use weaker sandbox for nested environments |

### hooks

Hook definitions (see the [Hooks Guide](hooks-guide.md) for details):

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write $CLAUDE_TOOL_INPUT_PATH"
          }
        ]
      }
    ]
  }
}
```

### enabledPlugins

Map of enabled plugins:

```json
{
  "enabledPlugins": {
    "my-plugin": true
  }
}
```

### Other Settings Fields

| Field | Description |
|---|---|
| `teammateMode` | Agent teams display mode: `auto`, `in-process`, `tmux` |
| `cleanupPeriodDays` | Session cleanup period (default: 30) |
| `respectGitignore` | Respect .gitignore for file operations (default: true) |
| `language` | Language for Claude's responses |
| `statusLine` | Custom status line text |
| `enableAllProjectMcpServers` | Auto-approve all project MCP servers |
| `alwaysThinkingEnabled` | Enable extended thinking |

---

## Environment Variables

These environment variables configure Claude Code's behavior:

### Authentication

| Variable | Purpose |
|---|---|
| `ANTHROPIC_API_KEY` | API key for authentication |
| `ANTHROPIC_AUTH_TOKEN` | Auth token (alternative to API key) |

### Model Configuration

| Variable | Purpose |
|---|---|
| `ANTHROPIC_MODEL` | Override the default model |
| `CLAUDE_CODE_SUBAGENT_MODEL` | Model for subagents |
| `CLAUDE_CODE_EFFORT_LEVEL` | Reasoning effort: `low`, `medium`, `high` |

### Behavior

| Variable | Purpose |
|---|---|
| `CLAUDE_CODE_DISABLE_AUTO_MEMORY` | Disable automatic memory saving |
| `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` | Enable agent teams (set to `1`) |

### Limits

| Variable | Purpose |
|---|---|
| `BASH_DEFAULT_TIMEOUT_MS` | Default timeout for bash commands (ms) |
| `CLAUDE_CODE_MAX_OUTPUT_TOKENS` | Max output tokens (default: 32000, max: 64000) |
| `MAX_MCP_OUTPUT_TOKENS` | Max MCP tool output tokens (default: 25000) |

### Disable Features

| Variable | Purpose |
|---|---|
| `DISABLE_AUTOUPDATER` | Disable automatic updates |
| `DISABLE_ERROR_REPORTING` | Disable error reporting |
| `DISABLE_TELEMETRY` | Disable telemetry |

---

## Full Example: Project Settings

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
    "deny": []
  },
  "env": {
    "NODE_ENV": "test"
  },
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write \"$CLAUDE_TOOL_INPUT_PATH\""
          }
        ]
      }
    ]
  }
}
```

## Full Example: User Settings

```json
{
  "permissions": {
    "allow": [
      "Read",
      "Glob",
      "Grep"
    ]
  },
  "model": "claude-opus-4-6"
}
```

## Full Example: Local Settings

```json
{
  "env": {
    "DATABASE_URL": "postgresql://localhost:5432/mydevdb",
    "API_SECRET": "dev-only-secret"
  }
}
```
