# Output Styles

Output styles modify Claude Code's system prompt to change how responses are formatted and structured. They control tone, verbosity, and instructional focus without affecting tool behavior.

## Built-in Styles

| Style | Behavior |
|-------|----------|
| **Default** | Standard software engineering focus — concise, code-oriented responses |
| **Explanatory** | Adds educational context — explains *why* behind suggestions, includes conceptual insights |
| **Learning** | Collaborative mode — adds `TODO(human)` markers for sections you should complete yourself, encourages learning by doing |

## How Styles Work

The default style includes two key system prompt components:
1. **Concise output instructions** — keeps responses brief
2. **Coding instructions** — software engineering best practices

When you switch styles:
- **Built-in styles** (Explanatory, Learning) exclude the concise output instructions, allowing longer responses
- **Custom styles** exclude *both* concise and coding instructions by default, replacing them with your custom prompt

To keep coding instructions in a custom style, set `keep-coding-instructions: true` in the frontmatter.

## Changing Styles

### Interactive Command

```
/output-style
```

Opens a picker to select from available styles. You can also specify directly:

```
/output-style explanatory
```

### Saved Preference

The active style is stored in `.claude/settings.local.json`:

```json
{
  "outputStyle": "explanatory"
}
```

This persists across sessions for the current project.

## Creating Custom Styles

Custom styles are Markdown files with YAML frontmatter.

### File Structure

```markdown
---
name: Reviewer
description: Focus on code review feedback
keep-coding-instructions: true
---

Provide detailed code review feedback. For each issue found:
1. State the problem clearly
2. Explain the impact
3. Suggest a specific fix with a code example

Categorize issues as: critical, warning, or suggestion.
```

### Frontmatter Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Display name shown in the style picker |
| `description` | Yes | Short description of the style's purpose |
| `keep-coding-instructions` | No | If `true`, retains default coding instructions alongside your custom prompt. Default: `false` |

### File Locations

| Location | Scope | Use Case |
|----------|-------|----------|
| `~/.claude/output-styles/` | User-level | Personal styles available in all projects |
| `.claude/output-styles/` | Project-level | Team-shared styles, committed to version control |

Project-level styles override user-level styles with the same name.

### Example: Minimal Style

```markdown
---
name: Concise
description: Ultra-short responses
---

Respond in as few words as possible. Use bullet points. No explanations unless asked.
```

### Example: Teaching Style

```markdown
---
name: Mentor
description: Socratic teaching approach
keep-coding-instructions: true
---

Instead of giving direct answers, guide the user with questions.
When showing code, explain each decision point.
Suggest exercises to reinforce concepts.
```

## Output Styles vs Other Customization

| Mechanism | Purpose | Scope |
|-----------|---------|-------|
| **Output Styles** | Controls response *format and tone* | Replaces parts of the system prompt |
| **CLAUDE.md** | Project rules and conventions | Always active alongside any style |
| **Subagents** | Isolated task execution | Runs in separate context, own instructions |
| **Skills** | Multi-step workflows | Triggered on demand, task-oriented |

Key distinction: CLAUDE.md instructions are *always* applied on top of whatever output style is active. Output styles control *how* Claude responds; CLAUDE.md controls *what rules* Claude follows.
