# Hooks: Deterministic Lifecycle Events

Hooks execute at specific points in Claude's lifecycle. Unlike CLAUDE.md instructions (which are advisory), hooks are deterministic -- they always execute and can block operations.

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

## Hook Events

| Event | When It Fires | Matcher | Can Block? |
|---|---|---|---|
| `SessionStart` | Session begins or resumes | `startup`, `resume`, `clear`, `compact` | No |
| `UserPromptSubmit` | Before Claude processes a user prompt | No matcher | Yes |
| `PreToolUse` | Before a tool executes | Tool name | Yes (allow/deny/ask) |
| `PermissionRequest` | Permission dialog appears | Tool name | Yes |
| `PostToolUse` | After a tool succeeds | Tool name | No |
| `PostToolUseFailure` | After a tool fails | Tool name | No |
| `Notification` | Notification sent | `permission_prompt`, `idle_prompt`, `auth_success`, `elicitation_dialog` | No |
| `SubagentStart` | Subagent spawned | Agent type name | No |
| `SubagentStop` | Subagent finishes | Agent type name | Yes |
| `Stop` | Claude finishes responding | None | Yes |
| `TeammateIdle` | Teammate about to idle | None | Yes (exit 2 only) |
| `TaskCompleted` | Task marked complete | None | Yes (exit 2 only) |
| `PreCompact` | Before context compaction | `manual`, `auto` | No |
| `SessionEnd` | Session terminates | `clear`, `logout`, `prompt_input_exit`, `bypass_permissions_disabled`, `other` | No |

### Exit Codes

- **0** -- Success. Hook passes.
- **2** -- Blocking error. stderr is fed back to Claude as context.
- **Other non-zero** -- Non-blocking error (logged but does not stop the operation).

### PreToolUse Special Returns

PreToolUse hooks can return JSON with:
- `permissionDecision`: `"allow"`, `"deny"`, or `"ask"` -- override the permission decision
- `updatedInput` -- modify the tool input before execution
- `additionalContext` -- inject additional context for Claude

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

## Environment Variables in Hooks

These environment variables are available inside hook commands:

| Variable | Available In | Description |
|---|---|---|
| `CLAUDE_TOOL_NAME` | PreToolUse, PostToolUse | Name of the tool being used |
| `CLAUDE_TOOL_INPUT` | PreToolUse, PostToolUse | JSON-encoded tool input |
| `CLAUDE_TOOL_INPUT_PATH` | PreToolUse, PostToolUse (Write/Edit) | File path from tool input |
| `CLAUDE_TOOL_OUTPUT` | PostToolUse | Tool output/result |
| `CLAUDE_SESSION_ID` | All events | Current session identifier |
| `CLAUDE_PROJECT_DIR` | All events | Root directory of the project |

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

### Auto-format with Prettier

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

### Auto-format with Black (Python)

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "black \"$CLAUDE_TOOL_INPUT_PATH\" 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

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

### Validate user prompts

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Check if this prompt asks Claude to delete production data. If so, reject it."
          }
        ]
      }
    ]
  }
}
```

---

## Hooks vs CLAUDE.md

| Aspect | Hooks | CLAUDE.md |
|---|---|---|
| Enforcement | Deterministic, always runs | Advisory, Claude may not follow |
| Mechanism | Shell commands, prompts, subagents | Natural language instructions |
| Use case | Linting, formatting, blocking, validation | Coding style, architecture guidance |
| Failure mode | Exit 2 blocks operation, stderr fed to Claude | Claude may ignore or misinterpret |
| Performance | Adds latency per event | No runtime cost |

**Rule of thumb:** Use hooks for things that must always happen (formatting, validation). Use CLAUDE.md for things Claude should know but can exercise judgment about (style preferences, architectural context).
