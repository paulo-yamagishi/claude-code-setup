# The Explore, Plan, Execute, Commit Workflow

Claude Code follows a four-phase workflow for every non-trivial task. Understanding and reinforcing this workflow produces the best results.

---

## Phase 1: Explore

**Read code before changing it.** This is the single most important habit.

Claude has access to powerful tools for understanding a codebase:

- **Glob** — Find files by name pattern (`**/*.ts`, `src/**/index.*`)
- **Grep** — Search file contents with regex (`function handleAuth`, `TODO.*security`)
- **Read** — Read specific files when you know the path

### What to explore

- The files you plan to modify
- Related files (imports, tests, types)
- Project structure and conventions
- Existing patterns for similar features

### Prompting for exploration

```
Look at the authentication module and understand how login works before making any changes.
```

```
Read the test files for the payment service so you understand the testing patterns used here.
```

**Anti-pattern:** Asking Claude to "add a logout endpoint" without first having it read the existing auth code. Claude will guess at patterns instead of following established ones.

---

## Phase 2: Plan

For non-trivial tasks, ask Claude to plan before executing.

### Using plan mode

You can type `/plan` or include planning instructions in your prompt:

```
Plan how you would refactor the database layer to support connection pooling. Don't make changes yet.
```

Claude will outline a step-by-step approach. Review it, ask questions, and approve before proceeding.

### When to plan

- Changes spanning multiple files
- Architectural decisions
- Unfamiliar codebases
- Anything where a mistake is costly to undo

### When to skip planning

- Single-file edits with clear intent
- Fixing a typo or obvious bug
- Running a command you've specified exactly

### Advanced Plan Mode Patterns

**Second-Claude plan review:** Spin up a separate Claude session, paste your plan, and ask it to critique the approach as a staff engineer. This catches blind spots before you invest time executing.

```
Here's my plan for refactoring the auth module: [paste plan]
Review this as a staff engineer. What did I miss? What would you push back on?
```

**Switch back to plan mode when things go sideways.** If execution is producing unexpected failures or the approach feels wrong, don't push through — return to plan mode and reassess.

```
/plan
This isn't working. Step back and reconsider the approach. What's a simpler way?
```

**Plan mode for verification, not just building.** After implementation, use plan mode to design your verification strategy before running tests.

```
/plan
The feature is implemented. Plan how to verify it works correctly — what tests to run, what edge cases to check, what to look for in the diff.
```

See `core/prompting-guide.md` for reset and verification prompt patterns.

---

## Phase 3: Execute

Once the plan is approved, Claude makes changes and verifies them.

### The execution loop

1. Make a change (Edit, Write)
2. Run relevant tests or linters
3. Fix any failures
4. Repeat until the change is complete and verified

### Key principles

- **Run tests after changes.** Don't assume edits are correct.
- **Make minimal changes.** Don't refactor unrelated code.
- **Follow existing patterns.** Match the style of surrounding code.
- **Iterate on failures.** If a test fails, read the error, fix it, and re-run.

### Prompting for execution

```
Go ahead with the plan. Run tests after each file change.
```

```
Make the changes we discussed. If any tests fail, fix them before moving on.
```

---

## Phase 4: Commit

After changes are verified, commit them properly.

### Commit rules

- **Stage specific files** — Use `git add <file>` not `git add -A` or `git add .`
- **Write descriptive messages** — Focus on "why" not "what"
- **Never force push** — Unless explicitly asked
- **Never amend previous commits** — Create new commits; amending can destroy work, especially after hook failures
- **Never skip hooks** — No `--no-verify` unless explicitly asked

### Prompting for commits

```
Commit these changes with a descriptive message.
```

```
Stage only the files we modified and commit.
```

**Anti-pattern:** After a pre-commit hook failure, using `--amend` on the next attempt. The failed commit never happened, so amending modifies the *previous* commit. Always create a new commit after fixing hook issues.

---

## Context Management

### When to use /clear

Use `/clear` to reset context between unrelated tasks. This prevents Claude from being confused by stale information from a previous task.

Good times to clear:
- Switching from one feature to another
- After completing a large task
- When Claude seems confused or stuck in a loop
- Before starting a fresh investigation

### Monitoring context

Use `/context` to see how much of the context window is consumed. When context is getting full, Claude will automatically compact, but you can proactively clear or start a new session.

### Parallel work with git worktrees

For working on multiple features simultaneously, use git worktrees instead of branches. Each worktree has its own working directory, so you can run separate Claude sessions without conflicts.

```bash
git worktree add ../my-feature-branch feature-branch
```

Then open a separate Claude session in that worktree directory.

---

## Anti-Patterns Summary

| Anti-Pattern | Why It Fails | What to Do Instead |
|---|---|---|
| Changing code without reading it | Claude guesses at patterns | Always explore first |
| Skipping tests | Broken code gets committed | Run tests after every change |
| Amending after hook failure | Destroys the previous commit | Create a new commit |
| Using `git add .` | Commits secrets or junk files | Stage specific files by name |
| Force pushing | Destroys remote history | Never force push unless asked |
| Huge context, no clearing | Claude loses track of the task | Use /clear between tasks |
| No plan for complex tasks | Incoherent multi-file changes | Plan first, execute second |
