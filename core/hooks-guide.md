# Hooks: Deterministic Lifecycle Events

Hooks execute at specific points in Claude's lifecycle. Unlike CLAUDE.md instructions (which are advisory), hooks are deterministic -- they always execute and can block operations.

For the full event table, exit codes, and environment variables, see [hooks-events-reference.md](hooks-events-reference.md).

---

## Hook Types

### command

Runs a shell command. Receives JSON on stdin with event context.

```json
{
  "type": "command",
  "command": "npx eslint --fix \"$CLAUDE_TOOL_INPUT_PATH\""
}
```

### prompt

A single-turn LLM evaluation. Returns `{ok: true/false, reason}` to approve or reject an operation.

```json
{
  "type": "prompt",
  "prompt": "Check if this code change follows our security guidelines."
}
```

### agent

Spawns a multi-turn subagent with tools (Read, Grep, Glob). Can run up to 50 turns.

```json
{
  "type": "agent",
  "prompt": "Review the changes made and check for security issues."
}
```

---

## Configuration

Hooks are defined in `.claude/settings.json` (or `.claude/settings.local.json` for personal hooks):

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Bash command about to run'"
          }
        ]
      }
    ],
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
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "terminal-notifier -message 'Claude finished' -title 'Claude Code'"
          }
        ]
      }
    ]
  }
}
```

Each event key contains an array of hook groups. Each group has an optional `matcher` (for tool-specific or event-specific matching) and a `hooks` array of hook definitions.

---

## Patterns

### Lint on save

Run a linter every time Claude writes or edits a file:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx eslint --fix \"$CLAUDE_TOOL_INPUT_PATH\" 2>/dev/null || true"
          }
        ]
      },
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "npx eslint --fix \"$CLAUDE_TOOL_INPUT_PATH\" 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

### Block dangerous commands

Prevent destructive bash commands:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "echo \"$CLAUDE_TOOL_INPUT\" | python3 -c \"import sys,json; cmd=json.load(sys.stdin)['command']; sys.exit(2 if any(d in cmd for d in ['rm -rf /','DROP DATABASE','--force']) else 0)\""
          }
        ]
      }
    ]
  }
}
```

## Async Hooks

Add `"async": true` to run a hook in the background without blocking Claude. The hook starts but Claude continues working immediately.

**When to use:** Long-running tasks like full test suites or linting large codebases.

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "npm test 2>&1 | tail -5",
            "async": true
          }
        ]
      }
    ]
  }
}
```

Async hooks cannot block operations (exit code 2 is ignored). Use them only for side effects like notifications or background checks.

---

### Notify on completion (macOS)

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude has finished\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```
