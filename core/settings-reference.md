# Settings Hierarchy and Configuration Reference

Claude Code uses a layered settings system. Higher-priority settings override lower ones.

For permission rules details, see [settings-permissions.md](settings-permissions.md). For sandbox, environment variables, and managed settings, see [settings-advanced.md](settings-advanced.md).

---

## Settings Scopes

| Priority | Scope | Location | Committed to Git |
|---|---|---|---|
| 1 | Managed | `managed-settings.json` | N/A |
| 2 | CLI flags | Command line arguments | No |
| 3 | Local | `.claude/settings.local.json` | No (gitignored) |
| 4 | Project | `.claude/settings.json` | Yes |
| 5 | User | `~/.claude/settings.json` | No |

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

Control which tools Claude can use. Permission rules use the format `Tool` or `Tool(specifier)`. Evaluation order: deny -> ask -> allow (first match wins). See [settings-permissions.md](settings-permissions.md) for full details.

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
    "defaultMode": "default"
  }
}
```

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
            "command": "npx prettier --write \"$CLAUDE_TOOL_INPUT_PATH\""
          }
        ]
      }
    ]
  }
}
```

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
