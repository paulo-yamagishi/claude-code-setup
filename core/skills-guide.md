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

## Skills vs Hooks

| | Skills | Hooks |
|---|--------|-------|
| **Trigger** | User-initiated or auto-loaded | Event-driven (tool use, session events, etc.) |
| **Interaction** | Interactive, Claude follows instructions | Automatic, runs deterministically |
| **Use case** | "Do this task for me" | "Always do this when X happens" |

For real-world skill examples, see [skills-examples.md](skills-examples.md).
