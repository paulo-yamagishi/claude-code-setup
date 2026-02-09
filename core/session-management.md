# Context and Session Management

## Context Window

Claude has a finite context window. Every file read, tool call, and message consumes context. Monitor usage with:

```
/context
```

This shows how much of the context window is used. When it gets full, Claude's ability to recall earlier parts of the conversation degrades.

## /clear — The Most Important Command

**Use `/clear` aggressively between unrelated tasks.** This is the single biggest productivity tip for Claude Code.

Every task leaves residue in context — file contents, tool outputs, intermediate reasoning. If you switch topics without clearing, Claude carries all that irrelevant information, which:

- Wastes context space
- Can confuse Claude with stale information
- Slows down responses

```
# Just finished fixing a bug in auth. Now want to add a new API endpoint.
/clear

# Fresh context for the new task
> add a GET /api/v2/users endpoint that returns paginated results
```

## /resume — Continue Previous Sessions

Resume a previous session with its full context:

```
/resume
```

Claude shows recent sessions and lets you pick one to continue. Useful when you need to come back to a task after a break.

## /compact — Manual Context Compaction

When context is getting large but you are not done with the current task:

```
/compact
```

This summarizes the conversation so far, keeping key information while freeing context space. Use it when `/context` shows high usage but you still have work to do in the current session.

## Git Worktrees for Parallel Sessions

Use git worktrees to run multiple Claude sessions on different branches simultaneously. Each worktree is a separate directory with its own working tree, so each Claude session has independent context.

```bash
# Create a worktree for a feature branch
git worktree add ../project-feature feature-branch

# Create a worktree for a bugfix
git worktree add ../project-bugfix bugfix-branch

# Run Claude in each directory — separate sessions, separate context
cd ../project-feature && claude
cd ../project-bugfix && claude
```

This is how you parallelize work with Claude Code. One session refactors the API while another fixes a UI bug — no context contamination.

```bash
# List active worktrees
git worktree list

# Remove when done
git worktree remove ../project-feature
```

### Power-User Worktree Patterns

For large projects, spin up 3-5 worktrees and keep them running. This is the natural extension of "one task per session" — each worktree is an isolated workspace with its own Claude session.

**Shell aliases for quick switching:**

```bash
# Add to your .bashrc / .zshrc
alias za='cd ~/worktrees/project-main && claude'
alias zb='cd ~/worktrees/project-feature && claude'
alias zc='cd ~/worktrees/project-bugfix && claude'
```

**Dedicated analysis worktree:**

Keep one read-only worktree for exploration — reviewing logs, querying data, or asking Claude to explain code — without polluting the working state of your active branches.

```bash
git worktree add ~/worktrees/project-analysis main
# Use this worktree for read-only exploration sessions
```

See `core/terminal-setup.md` for organizing tabs and windows across multiple worktrees.

## Session Strategies

### One Task Per Session (Recommended)

Start Claude, do one focused task, exit. This gives Claude maximum context for the task and avoids cross-contamination.

```bash
# Session 1: fix the bug
claude "fix the null pointer in UserService.getProfile"

# Session 2: add the feature
claude "add rate limiting to the /api/search endpoint"
```

### Multiple Tasks with /clear

If you prefer staying in one session, use `/clear` between tasks:

```
> fix the failing test in test_auth.py
# ... Claude fixes it ...

/clear

> now add input validation to the signup form
# ... fresh context for this task ...
```

### Large Refactors: Plan Then Execute

For big changes, split into two phases:

**Planning session**: Explore the codebase, understand the current architecture, create a plan.

```
> I want to migrate from REST to GraphQL. Explore the codebase and create
  a migration plan with specific steps.
```

**Execution sessions**: Take each step from the plan and execute it in a focused session.

```
> Step 1 from the migration plan: set up the GraphQL server and schema
  for the User type. Here's the plan: [paste relevant section]
```

## Context Optimization

### Keep CLAUDE.md Concise

CLAUDE.md is loaded into every session. Keep it under 300 lines. Include only what Claude needs for every task:

- Build and test commands
- Code conventions
- Project structure overview
- Key architectural decisions

Move detailed information to separate files and reference them with `@imports`.

### Use @imports for Detailed Content

Instead of putting everything in CLAUDE.md:

```markdown
# CLAUDE.md

@docs/api-conventions.md
@docs/database-schema.md

## Quick Reference
- Run tests: `pytest`
- Run dev server: `npm run dev`
```

The imported files are loaded only when relevant, saving context for other work.

### Limit MCP Servers

Each MCP server adds tool descriptions to the context. If you have 5 MCP servers with 10 tools each, that is 50 tool descriptions consuming context before you even start working.

Only enable the MCP servers you actually need for the current project.

### Let Claude Read Files

Don't paste large files into the chat. Let Claude read them with the Read tool:

```
# Bad — wastes context with the full file
> here's my 500-line config file: [paste]

# Good — Claude reads only what it needs
> check the webpack config for any issues with the build
```

Claude will read the file, and the content is managed more efficiently in context.

## Auto-Compaction

When context reaches approximately 80% capacity, Claude automatically compacts the conversation. This summarizes earlier messages while preserving key information.

You do not need to manage this manually — it happens transparently. But if you notice Claude forgetting earlier details in a long session, it may be a sign that auto-compaction has occurred and some nuance was lost. In that case, start a fresh session.

## PreCompact and PostCompact Hooks

You can customize what happens during compaction with hooks in your settings:

- **PreCompact**: Runs before compaction. Use it to save important state that might be lost during summarization.
- **PostCompact**: Runs after compaction. Use it to re-inject critical context that must survive compaction.

These are advanced features for users who find that compaction is losing information they need to preserve.
