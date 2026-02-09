# How to Write Effective CLAUDE.md Files

CLAUDE.md is the first thing Claude reads when it starts a session. It serves as project memory -- telling Claude what it needs to know about your codebase, conventions, and workflows.

---

## Memory Hierarchy

Claude reads instructions from multiple sources:

| Type | Location | Shared With |
|---|---|---|
| Managed policy | `/Library/Application Support/ClaudeCode/CLAUDE.md` (macOS) | All org users |
| Project memory | `./CLAUDE.md` or `./.claude/CLAUDE.md` | Team (git) |
| Project rules | `./.claude/rules/*.md` | Team (git) |
| User memory | `~/.claude/CLAUDE.md` | Just you (all projects) |
| Project local | `./CLAUDE.local.md` | Just you (current project) |
| Auto memory | `~/.claude/projects/<project>/memory/` | Just you (per project) |

### Managed policy

Organization-wide instructions set by administrators. Located at `/Library/Application Support/ClaudeCode/CLAUDE.md` on macOS. Cannot be overridden.

### Project CLAUDE.md

The main file at the repository root (`./CLAUDE.md`) or inside the `.claude` directory (`./.claude/CLAUDE.md`). Committed to version control and shared with the team.

### .claude/rules/*.md

Modular rule files that are automatically imported. Use these to split large CLAUDE.md files into focused topics:

```
.claude/rules/
  testing.md
  code-style.md
  api-conventions.md
  deployment.md
```

Every `.md` file in this directory is auto-imported -- no explicit reference needed. Rules support path-specific YAML frontmatter with a `paths` field and glob patterns to scope rules to specific files.

### User ~/.claude/CLAUDE.md

Your personal global defaults. Good for preferences that apply across all projects (e.g., "I prefer concise explanations" or "Always use TypeScript strict mode").

### CLAUDE.local.md

Personal overrides for a specific project at `./CLAUDE.local.md`. Not committed to git, so it won't affect teammates. Good for local paths, personal tool preferences, or dev environment specifics.

### Auto memory

Auto-generated memory files stored at `~/.claude/projects/<project>/memory/`. Claude writes these to remember decisions and patterns across sessions for a specific project.

---

## @import Syntax

Reference other files from any CLAUDE.md file using the `@` prefix:

```markdown
@docs/architecture.md
@scripts/README.md
@~/.claude/my-instructions.md
```

The referenced file's contents are pulled into context. Use this to point Claude at detailed documentation without duplicating it in CLAUDE.md.

Maximum recursive import depth: **5 hops**.

---

## Best Practices

### Keep it concise

Aim for under 300 lines. Claude reads the entire file at session start, and overly long files waste context and dilute important rules.

### Put critical rules first

Claude processes top-down. The most important instructions should be at the top of the file. If you have a rule that must never be violated, put it in the first section.

### Use @imports for detail

Instead of pasting your entire API design guide into CLAUDE.md, write:

```markdown
## API Design
Follow the conventions in @docs/api-conventions.md
```

### Split rules into .claude/rules/

For projects with many conventions, use modular rule files instead of one massive CLAUDE.md:

```markdown
# Instead of a 500-line CLAUDE.md, create:
.claude/rules/testing.md      # Test conventions
.claude/rules/style.md        # Code style rules
.claude/rules/git.md          # Git workflow rules
```

### What to include

- Project architecture overview (key directories, entry points)
- Coding style rules (naming, formatting, patterns)
- Test commands (`npm test`, `pytest`, `cargo test`)
- Common patterns ("we use repository pattern for data access")
- Build and run commands
- Important gotchas or non-obvious conventions

### What not to include

- Obvious things ("use descriptive variable names")
- Language basics ("TypeScript uses interfaces")
- Verbose explanations of well-known patterns
- Information that changes frequently (put in @imported docs instead)

---

## Structure Template

```markdown
# Project Name

## Overview
Brief description of what this project does and its tech stack.

## Architecture
- `src/api/` -- API routes and controllers
- `src/services/` -- Business logic
- `src/models/` -- Database models
- `src/utils/` -- Shared utilities

## Commands
- Build: `npm run build`
- Test: `npm test`
- Lint: `npm run lint`
- Dev server: `npm run dev`

## Code Style
- Use TypeScript strict mode
- Prefer named exports over default exports
- Use async/await, never raw promises
- Error handling: use Result types, not try/catch

## Testing
- Unit tests go next to source files: `foo.ts` -> `foo.test.ts`
- Use `describe/it` blocks, not `test()`
- Mock external services, never hit real APIs in tests

## Conventions
- API responses follow: `{ data: T, error?: string }`
- Database migrations in `src/migrations/`, run with `npm run migrate`
- Environment variables defined in `.env.example`
```

---

## Good vs Bad Examples

### Good entries

```markdown
## Testing
Run tests with `pytest -x --tb=short`. Always run tests before committing.
Test files live in `tests/` mirroring the `src/` structure.
Use `factory_boy` for test data, never create objects manually.
```

```markdown
## Style
We use Black for formatting (line length 100). Do not manually format code.
Type hints are required on all public functions.
```

### Bad entries

```markdown
## Testing
Testing is very important for software quality. Tests help us catch bugs
before they reach production. There are many types of tests including
unit tests, integration tests, and end-to-end tests. We primarily use
unit tests in this project...
```
This is too verbose. Claude knows what testing is. Tell it *what to do*, not *why testing matters*.

```markdown
## Style
Use good variable names. Write clean code. Follow best practices.
```
This is too vague. Every project wants "clean code." Specify the actual conventions: formatter, line length, naming patterns, import ordering.

```markdown
## Architecture
This is a web application.
```
This tells Claude nothing useful. Specify the framework, directory structure, entry points, and key patterns.

---

## Maintaining CLAUDE.md Over Time

CLAUDE.md should evolve as your project evolves. When Claude makes a mistake or you correct its behavior, have it write a rule — but route it to the right place to avoid bloating the main file.

### Where rules should go

| Rule Type | Destination | Example |
|---|---|---|
| Short, universal | `CLAUDE.md` directly | "Use `pnpm`, not `npm`" |
| Detailed, topic-specific | `.claude/rules/topic.md` | Full testing conventions |
| Per-project learnings | Auto memory (`~/.claude/projects/<project>/memory/`) | "This repo uses a custom ORM" |
| Task-specific notes | `@notes/task-name.md` referenced via `@import` | Migration plan details |

### The prompt to use

After correcting Claude, tell it:

```
Add what you just learned to the appropriate place — CLAUDE.md for short rules,
.claude/rules/ for detailed conventions, auto memory for project-specific learnings.
```

Claude writes effective rules for itself. Leverage this, but curate regularly.

### Maintenance cadence

- **Weekly:** Scan CLAUDE.md for stale entries. Remove rules for patterns that are now second nature.
- **After corrections:** Route the new rule to the right destination. Keep CLAUDE.md under 300 lines.
- **Monthly:** Promote verbose CLAUDE.md entries to `.claude/rules/` files. Consolidate similar rules.

The goal is a tight, high-signal CLAUDE.md that stays effective as your project grows — not an append-only log of every lesson learned.
