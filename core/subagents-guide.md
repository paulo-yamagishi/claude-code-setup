# Custom Agents and Built-in Subagent Types

## What Are Subagents?

Subagents are isolated Claude instances spawned via the Task tool. They run independently from the main conversation, execute their work, and return results as a single message. This isolation keeps the main context clean and allows parallel execution of independent tasks.

## Built-in Subagent Types

Claude Code includes several specialized subagent types optimized for common tasks.

### Explore

A fast codebase exploration agent using haiku. Has read-only access with Glob, Grep, and Read tools. Cannot modify files. Designed for quickly finding information across a codebase.

Use for: finding function definitions, understanding code structure, locating configuration files, answering questions about how something works.

### Plan

An architecture and planning agent that inherits the current model. Has read-only tools. Analyzes code, considers tradeoffs, and produces structured plans without making any changes.

Use for: designing new features, evaluating refactoring strategies, creating implementation plans.

### General-Purpose

A full-capability agent that inherits the current model. Has access to all tools. Handles complex tasks that require reading, writing, and executing code.

Use for: implementing features, fixing bugs, performing multi-step tasks that combine research and modification.

### Bash

A command execution specialist that inherits the current model. Has access to terminal/bash tools. Optimized for running shell commands, scripts, and CLI operations.

Use for: running test suites, executing build commands, interacting with system tools.

## Custom Subagents

Define custom agents by creating Markdown files in `.claude/agents/`.

### File Location

```
.claude/agents/agent-name.md
```

### File Format

```markdown
---
name: code-reviewer
description: Reviews code changes for quality and security
tools:
  - Read
  - Glob
  - Grep
  - Bash
model: sonnet
permissionMode: acceptEdits
maxTurns: 20
---

# Code Reviewer Agent

You are a code review specialist. Review the provided changes for:
- Code quality and readability
- Security vulnerabilities
- Performance issues
- Test coverage

For each issue found, report:
1. File and line number
2. Severity (critical, warning, info)
3. Description of the issue
4. Suggested fix
```

### Configuration Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Identifier for the agent (required). |
| `description` | string | Short summary of what the agent does (required). |
| `tools` | list | Allowlist of tools the agent can use. Inherits all tools if omitted. Supports `Task(agent_type)` for nested agents. |
| `disallowedTools` | list | Denylist of tools the agent cannot use. |
| `model` | string | Model to use: `sonnet`, `opus`, `haiku`, `inherit` (default: `inherit`). |
| `permissionMode` | string | Permission handling: `default`, `acceptEdits`, `delegate`, `dontAsk`, `bypassPermissions`, `plan`. |
| `maxTurns` | number | Maximum agentic turns the agent can take. |
| `skills` | list | Skills preloaded when the agent starts. |
| `mcpServers` | list/object | MCP server names or inline definitions available to the agent. |
| `hooks` | object | Lifecycle hooks scoped to this agent. |
| `memory` | string | Memory scope: `user`, `project`, or `local` (persistent cross-session). |

## Memory Persistence

Agents can maintain state across sessions using persistent memory. Configure this with the `memory` field in the agent definition. Memory locations:

| Scope | Location |
|---|---|
| `user` | `~/.claude/agent-memory/<name>/` |
| `project` | `.claude/agent-memory/<name>/` |
| `local` | `.claude/agent-memory-local/<name>/` |

The first 200 lines of `MEMORY.md` are loaded at agent startup.

```markdown
---
name: project-expert
description: Maintains deep knowledge about the project
tools:
  - Read
  - Glob
  - Grep
memory: project
---
```

## Context Isolation

Subagents do **not** share context with the parent conversation. They start with only the prompt you provide. Their entire output is returned to the parent as a single message.

This means:
- The subagent cannot see earlier conversation history.
- The parent does not see the subagent's intermediate steps.
- You must include all necessary context in the prompt you send to the subagent.

## Background Subagents

Subagents can run concurrently in the background. Background subagents have their permissions pre-approved upfront. Any unapproved permissions are auto-denied. Background subagents cannot use MCP tools.

Use `Ctrl+B` to background a running subagent.

## Parallel Execution

Launch multiple agents simultaneously for independent tasks. Use the `run_in_background` parameter to spawn agents without blocking the main conversation.

```
Example workflow:
1. Spawn agent A to research the authentication module (background)
2. Spawn agent B to research the database schema (background)
3. Spawn agent C to research the API routes (background)
4. Collect results from all three
5. Synthesize findings in the main conversation
```

## The Task Tool Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `description` | string | Short label for the task (shown in status). |
| `prompt` | string | The full prompt sent to the subagent. |
| `subagent_type` | string | Built-in type to use (e.g., "Explore", "Bash", "Plan"). |
| `model` | string | Model override (sonnet, opus, haiku). |
| `run_in_background` | boolean | If true, the task runs without blocking. |
| `resume` | string | Resume a previously started background task by ID. |
| `max_turns` | number | Maximum number of tool-use turns the agent can take. |

## When to Use Subagents

**Good use cases:**
- Complex multi-step research across many files (keeps main context clean).
- Parallel independent tasks (e.g., researching three different modules simultaneously).
- Tasks that produce large intermediate output you do not need in the main conversation.
- Specialized work that benefits from a focused agent with constrained tools.

**When NOT to use:**
- Simple file reads or single grep searches -- the overhead is not worth it.
- Tasks that need the main conversation's context or history.
- Tasks where you need to interactively guide the agent mid-execution.
- Trivial operations that complete in one or two tool calls.

## Practical Subagent Tips

### Prompting for subagent use

When a task needs more compute or would benefit from parallel exploration, append "use subagents" to your request:

```
Research how authentication works across all three services. Use subagents.
```

This signals Claude to spawn parallel agents instead of doing everything sequentially in the main context. Use this as a prompting shortcut, not a default — the existing guidance on when NOT to use subagents still applies.

### Keeping the main context clean

Offload individual research or implementation tasks to subagents whenever the intermediate output would bloat your main conversation. For example, if you need to understand three modules before making a decision, spawn three Explore agents in parallel rather than reading all three sequentially in the main context.

### Permission routing with hooks

Use a `PermissionRequest` hook to auto-approve safe subagent actions (file reads, grep, glob) while requiring confirmation for writes and bash commands. This lets subagents work autonomously for research tasks without constant permission prompts.

```json
{
  "hooks": {
    "PermissionRequest": [
      {
        "matcher": "Read|Glob|Grep",
        "command": "echo approve"
      }
    ]
  }
}
```

See `core/hooks-guide.md` for full hook configuration details.

## Example Custom Agents

### Security Auditor

```markdown
---
name: security-auditor
description: Scans code for security vulnerabilities
tools:
  - Read
  - Glob
  - Grep
model: sonnet
---

# Security Auditor

Scan the codebase for common security vulnerabilities:
- SQL injection
- XSS (cross-site scripting)
- Hardcoded secrets and credentials
- Insecure dependencies
- Missing input validation
- Improper error handling that leaks information

Report findings with severity, location, and remediation steps.
```

### Test Writer

```markdown
---
name: test-writer
description: Generates unit tests for specified code
tools:
  - Read
  - Write
  - Glob
  - Grep
  - Bash
model: sonnet
permissionMode: acceptEdits
---

# Test Writer

You write thorough unit tests. Given a file or function:
1. Read and understand the code.
2. Identify edge cases, error conditions, and happy paths.
3. Write tests following the project's existing test patterns.
4. Run the tests to verify they pass.
```
