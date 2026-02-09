# Advanced Settings: Sandbox, Environment Variables, and More

For the settings hierarchy and basic schema, see [settings-reference.md](settings-reference.md). For permission rules, see [settings-permissions.md](settings-permissions.md).

---

## Sandbox Configuration

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

---

## Environment Variables in Settings

```json
{
  "env": {
    "NODE_ENV": "development",
    "DATABASE_URL": "postgresql://localhost:5432/mydb"
  }
}
```

## Environment Variables for Claude Code

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

## Managed Settings

Set by enterprise administrators. Cannot be overridden by any other scope.

Managed settings file locations:

| Platform | Path |
|---|---|
| macOS | `/Library/Application Support/ClaudeCode/managed-settings.json` |
| Linux | `/etc/claude-code/managed-settings.json` |
| Windows | `C:\Program Files\ClaudeCode\managed-settings.json` |

---

## Other Settings Fields

| Field | Description |
|---|---|
| `teammateMode` | Agent teams display mode: `auto`, `in-process`, `tmux` |
| `cleanupPeriodDays` | Session cleanup period (default: 30) |
| `respectGitignore` | Respect .gitignore for file operations (default: true) |
| `language` | Language for Claude's responses |
| `statusLine` | Custom status line text |
| `enableAllProjectMcpServers` | Auto-approve all project MCP servers |
| `alwaysThinkingEnabled` | Enable extended thinking |
| `enabledPlugins` | Map of enabled plugins |
| `additionalDirectories` | Extend the set of working directories |
