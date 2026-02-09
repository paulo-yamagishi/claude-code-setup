# Hook Events Reference

For hook concepts, configuration, and patterns, see [hooks-guide.md](hooks-guide.md).

---

## Event Table

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

## Exit Codes

- **0** -- Success. Hook passes.
- **2** -- Blocking error. stderr is fed back to Claude as context.
- **Other non-zero** -- Non-blocking error (logged but does not stop the operation).

## PreToolUse Special Returns

PreToolUse hooks can return JSON with:
- `permissionDecision`: `"allow"`, `"deny"`, or `"ask"` -- override the permission decision
- `updatedInput` -- modify the tool input before execution
- `additionalContext` -- inject additional context for Claude

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

## Hooks vs CLAUDE.md

| Aspect | Hooks | CLAUDE.md |
|---|---|---|
| Enforcement | Deterministic, always runs | Advisory, Claude may not follow |
| Mechanism | Shell commands, prompts, subagents | Natural language instructions |
| Use case | Linting, formatting, blocking, validation | Coding style, architecture guidance |
| Failure mode | Exit 2 blocks operation, stderr fed to Claude | Claude may ignore or misinterpret |
| Performance | Adds latency per event | No runtime cost |

**Rule of thumb:** Use hooks for things that must always happen (formatting, validation). Use CLAUDE.md for things Claude should know but can exercise judgment about (style preferences, architectural context).
