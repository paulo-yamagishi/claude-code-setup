# Best Practices for Claude Code

A distilled guide to getting the most out of Claude Code, organized by impact.

---

## 1. Manage Your Context Window

The context window is your most valuable resource. Every token of conversation history competes for attention.

- **Use `/clear` between unrelated tasks** — a fresh context produces better results than a cluttered one
- **Use `/compact` when context grows large** — summarizes history while preserving key details
- **Delegate investigations to subagents** — they get their own context window and return only the answer
- **Be specific in prompts** — vague requests force Claude to load more files to understand what you want

### Signs your context is overloaded

- Claude forgets instructions from earlier in the conversation
- Responses become slower and less focused
- Claude re-reads files it already examined

---

## 2. Give Claude Verification Tools

Claude works best when it can check its own work. Always provide a way to verify.

```
# Good — Claude can run and verify
Add a function to parse ISO dates. There are existing tests in tests/date_utils_test — add cases there too.

# Less effective — no verification path
Add a function to parse ISO dates.
```

**Effective verification methods:**

| Method | When to use |
|--------|-------------|
| Test suites | Code changes with existing test infrastructure |
| Build commands | Compilation or type-checking after changes |
| Screenshots | UI changes (paste screenshots into the conversation) |
| Expected output | Data transformations, scripts, formatting changes |
| Linters/formatters | Style and correctness checks via hooks |

### Use hooks for automatic verification

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Verify before executing'"
          }
        ]
      }
    ]
  }
}
```

---

## 3. Follow the Four-Phase Workflow

For non-trivial tasks, guide Claude through four phases. See `workflow.md` for full details.

1. **Explore** — Read code before changing it. Point Claude at relevant files and patterns.
2. **Plan** — Ask Claude to outline its approach before writing code. Review the plan.
3. **Execute** — Let Claude implement the plan. It works best with a clear target.
4. **Commit** — Use `/commit` or let hooks handle formatting and tests.

```
# Phase 1: Explore
Read the auth module in src/auth/ and understand how sessions work.

# Phase 2: Plan
Now plan how to add refresh token support. List the files you'll change and why.

# Phase 3: Execute
Implement the plan.
```

---

## 4. Write Effective Prompts

### Be specific about scope

```
# Good — clear scope
Fix the null pointer in UserService.getProfile() when the user has no avatar set.

# Vague — Claude must guess
Fix the user profile bug.
```

### Point to sources and patterns

```
# Good — provides patterns
Add a new API endpoint for /api/v2/orders following the same pattern as /api/v2/products.
Look at src/routes/products.ts for the pattern.

# Less effective
Add an orders endpoint.
```

### Let Claude interview you

For ambiguous requirements, ask Claude to ask questions first:

```
I want to add caching to the API. Before implementing anything, ask me clarifying questions about requirements.
```

---

## 5. Configure Your Environment

A well-configured project reduces friction for every task.

### CLAUDE.md

Place project instructions where Claude can find them:

- `CLAUDE.md` in project root — global instructions
- `CLAUDE.md` in subdirectories — scoped instructions for that area
- `~/.claude/CLAUDE.md` — personal preferences across all projects

Keep CLAUDE.md focused: build commands, test commands, code conventions, architecture notes. See `core/claude-md-guide.md`.

### Permissions

Set `permissions.allow` in `.claude/settings.json` to reduce approval prompts:

```json
{
  "permissions": {
    "allow": [
      "Bash(npm test)",
      "Bash(npm run lint)",
      "Bash(npm run build)"
    ]
  }
}
```

### Hooks, Skills, and MCP

- **Hooks** — Automate checks on tool events. See `core/hooks-guide.md`.
- **Skills** — Reusable prompt templates for common workflows. See `core/skills-guide.md`.
- **MCP servers** — Connect external tools (databases, APIs, design tools). See `core/mcp-guide.md`.

---

## 6. Manage Sessions Effectively

| Command | Purpose |
|---------|---------|
| `/clear` | Reset context between tasks |
| `/compact` | Compress context when it grows large |
| `/rewind` | Undo the last turn if Claude went wrong |
| `/model` | Switch models mid-session |

### Use subagents for investigation

When you need to research something without polluting your main context:

```
Use a subagent to find all places where we handle authentication errors,
then summarize the patterns you find.
```

---

## 7. Automate and Scale

### Headless mode

Run Claude non-interactively for batch operations:

```bash
claude -p "Update copyright headers in all source files" --allowedTools 'Edit' 'Glob' 'Grep' 'Read'
```

### Parallel sessions

Run multiple Claude sessions on independent tasks:

```bash
# In separate terminals
claude -p "Add input validation to the user API" &
claude -p "Write tests for the payment module" &
```

### Multi-agent fan-out

Use agent teams for large-scale tasks. A lead agent delegates subtasks to worker agents that execute in parallel. See `core/agent-teams-guide.md`.

---

## 8. Common Failure Patterns

### Kitchen-sink session

**Problem:** One long session covering many unrelated tasks. Context gets overloaded.

**Fix:** Use `/clear` between tasks. One task per session when possible.

### Over-specified CLAUDE.md

**Problem:** CLAUDE.md is hundreds of lines long with edge cases and exceptions. Claude spends tokens processing instructions instead of doing work.

**Fix:** Keep CLAUDE.md concise. Move detailed instructions into subdirectory CLAUDE.md files where they are relevant.

### Trust-then-verify gap

**Problem:** Accepting Claude's output without verification, then finding bugs later.

**Fix:** Always provide verification. Run tests, check types, review diffs. Use pre-commit hooks to automate checks.

### No exploration phase

**Problem:** Jumping straight to "implement X" without letting Claude read the codebase first.

**Fix:** Start with exploration. Ask Claude to read relevant code before making changes.

### Ignoring the plan

**Problem:** Letting Claude execute without reviewing its plan. It may take an over-engineered approach.

**Fix:** Ask Claude to plan first. Review the plan. Redirect if needed.
