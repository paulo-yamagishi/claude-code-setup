# Creating Skills with SKILL.md

## What Are Skills?

Skills are reusable, prompt-based capabilities that you invoke with `/skill-name` in Claude Code. They let you package a set of instructions for a specific task so you can trigger them on demand rather than re-explaining what you want every time.

When you type `/fix-issue 42`, Claude loads the skill's instructions, substitutes the arguments, and executes accordingly. Skills are the primary mechanism for extending Claude Code with your own domain-specific workflows.

## SKILL.md File Format

A skill is defined by a single `SKILL.md` file containing YAML frontmatter followed by Markdown instructions.

```markdown
---
name: fix-issue
description: Fix a GitHub issue by number
argument-hint: "[issue-number]"
allowed-tools:
  - Bash(gh issue view *)
  - Read
  - Write
  - Edit
---

# Fix Issue Skill

You are tasked with fixing a GitHub issue.

1. Fetch the issue details using `gh issue view $ARGUMENTS`.
2. Read the issue description and any comments to understand the problem.
3. Search the codebase for relevant files.
4. Implement the fix.
5. Run tests to verify the fix.
6. Create a commit with a message referencing the issue.
```

## Frontmatter Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Identifier used to invoke the skill. Lowercase, hyphens only, max 64 chars. |
| `description` | string | Short summary of when to use the skill. Claude uses this for auto-loading. |
| `argument-hint` | string | Autocomplete hint shown to users, e.g. `[issue-number]`. |
| `disable-model-invocation` | boolean | If `true`, the skill can only be invoked manually via `/name`, not auto-loaded by Claude. |
| `user-invocable` | boolean | If `false`, the skill is hidden from the `/` menu and can only be invoked by Claude. |
| `allowed-tools` | list | Tools that can run without permission prompts when this skill is active. |
| `model` | string | Model override for this skill. |
| `context` | string | Set to `fork` to run the skill in a forked subagent. |
| `agent` | string | Subagent type to use when `context: fork`. |
| `hooks` | object | Lifecycle hooks scoped to this skill. |

## String Substitutions

Skills support these substitutions in the Markdown body:

| Substitution | Description |
|---|---|
| `$ARGUMENTS` | The full argument string passed after the skill name |
| `$ARGUMENTS[N]` | The Nth argument (0-indexed) |
| `$N` | Shorthand for `$ARGUMENTS[N]` |
| `${CLAUDE_SESSION_ID}` | The current session ID |

## Dynamic Context

Use `!`command`` syntax to run shell commands and include their output as dynamic context in the skill body:

```markdown
Current branch: !`git branch --show-current`
Recent commits: !`git log --oneline -5`
```

## File Locations

Skills can be defined at multiple levels:

- **Enterprise (managed):** Distributed via managed settings/plugins.
- **User-level:** `~/.claude/skills/skill-name/SKILL.md` -- available across all projects.
- **Project-level:** `.claude/skills/skill-name/SKILL.md` -- available only within that project.
- **Plugin skills:** Namespaced as `/plugin-name:skill-name`.

Project-level skills take precedence over user-level skills with the same name.

## Invoking Skills

Type `/skill-name` followed by any arguments:

```
/fix-issue 42
/create-component Button
```

## Best Practices

**Keep skills focused on one task.** A skill called `fix-issue` should fix an issue -- it should not also deploy, update docs, and send a Slack notification. Compose multiple skills if you need a larger workflow.

**Include clear instructions.** The Markdown body is the prompt Claude follows. Be specific about steps, tools to use, and expected outcomes.

**Use $ARGUMENTS for parameterization.** Do not hardcode values that vary between invocations.

**Provide good descriptions.** The `description` field is used by Claude to decide when to auto-load skills. Make it descriptive.

**Test your skills.** Invoke them with different arguments and edge cases to make sure the instructions produce correct behavior.

## Skills vs CLAUDE.md

| | Skills | CLAUDE.md |
|---|--------|-----------|
| **Activation** | On-demand via `/` command or auto-loaded by Claude | Always loaded at start of conversation |
| **Purpose** | Task-specific workflows | Project-wide context, conventions, rules |
| **Scope** | Narrow, single task | Broad, applies to everything |
| **Parameterized** | Yes, via $ARGUMENTS | No |

Use CLAUDE.md for things Claude should always know (coding standards, project structure). Use skills for specific actions you trigger when needed.

## Real-World Skill Examples

Beyond the basic examples below, here are patterns from power users.

### Tech Debt Scanner

Run at the end of every session to catch accumulated debt:

```markdown
---
name: techdebt
description: Scan codebase for duplicated code and tech debt patterns
allowed-tools:
  - Grep
  - Glob
  - Read
---

# Tech Debt Scan

Scan the codebase for:
1. Duplicated code blocks (3+ similar implementations)
2. TODO/FIXME/HACK comments added in the last week
3. Functions over 100 lines
4. Files with no test coverage

Report findings ranked by severity. Suggest concrete refactoring steps for the top 3 issues.
```

### Context Sync

Aggregate context from multiple tools into one dump before starting work:

```markdown
---
name: context-sync
description: Pull context from Slack, GitHub, and project docs
allowed-tools:
  - Bash(gh *)
  - Read
  - Glob
---

# Context Sync

Gather current project context:
1. Recent GitHub issues and PRs: `gh issue list --limit 10` and `gh pr list --limit 10`
2. Read the project CHANGELOG and recent commit messages
3. Summarize: what's in progress, what's blocked, what shipped recently

Present a brief status report I can scan in 30 seconds.
```

This pairs well with MCP servers for Slack, Google Drive, or Asana — see `core/mcp-guide.md` for setup.

### Analytics Engineer

For data teams using dbt or similar tools:

```markdown
---
name: analytics
description: Write and test dbt models
argument-hint: "[model-name]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash(dbt *)
  - Glob
---

# Analytics Engineer

Write or update the dbt model for $ARGUMENTS.
1. Read existing models in `models/` for conventions.
2. Write the model SQL following project patterns.
3. Run `dbt compile` to validate.
4. Run `dbt test` on the new model.
5. Update the schema YAML if adding new columns.
```

## Skills vs Hooks

| | Skills | Hooks |
|---|--------|-------|
| **Trigger** | User-initiated or auto-loaded | Event-driven (tool use, session events, etc.) |
| **Interaction** | Interactive, Claude follows instructions | Automatic, runs deterministically |
| **Use case** | "Do this task for me" | "Always do this when X happens" |

## Example Skills

### Code Review

```markdown
---
name: code-review
description: Review staged changes for quality issues
allowed-tools:
  - Bash(git diff *)
  - Read
  - Glob
  - Grep
---

# Code Review

Review all staged changes (`git diff --cached`) for:
- Logic errors and bugs
- Security vulnerabilities
- Performance problems
- Code style violations
- Missing error handling

Provide a summary with severity ratings (critical, warning, info).
```

### Create Component

```markdown
---
name: create-component
description: Create a new React component with tests
argument-hint: "[ComponentName]"
allowed-tools:
  - Write
  - Read
  - Glob
---

# Create Component

Create a new React component called $ARGUMENTS.

1. Create `src/components/$ARGUMENTS/$ARGUMENTS.tsx`.
2. Create `src/components/$ARGUMENTS/$ARGUMENTS.test.tsx`.
3. Create `src/components/$ARGUMENTS/index.ts` with a re-export.
4. Follow existing component patterns in the codebase.
```

### Deploy

```markdown
---
name: deploy
description: Deploy the application to a specified environment
argument-hint: "[environment]"
allowed-tools:
  - Bash(./scripts/deploy.sh *)
  - Bash(npm run build)
  - Bash(npm test)
---

# Deploy

Deploy the application to $0.

1. Run the test suite.
2. Build the application for the target environment.
3. Run the deploy script: `./scripts/deploy.sh $0`.
4. Verify the deployment by checking the health endpoint.
5. Report the deployment status.
```
