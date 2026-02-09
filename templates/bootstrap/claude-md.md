# CLAUDE.md Template for /bootstrap

Customize all `[bracketed placeholders]` based on detected stack.

```markdown
# CLAUDE.md — Project Instructions

## Overview
[Brief description based on package.json description, README.md, or inferred purpose]

## Architecture

### Directory Structure
- `src/` — [detected purpose]
- `tests/` — [test framework] test files
- `config/` — Configuration files
- [other detected directories]

### Key Files
- [Entry points: main.py, index.ts, main.go, etc.]
- [Config files: package.json, pyproject.toml, etc.]

## Commands

### Development
` ``bash
[Detected dev command: npm run dev, python manage.py runserver, cargo run, etc.]
` ``

### Testing
` ``bash
[Detected test command: npm test, pytest, cargo test, go test ./..., etc.]
` ``

### Linting
` ``bash
[Detected lint command: npm run lint, ruff check ., cargo clippy, golangci-lint run, etc.]
` ``

### Formatting
` ``bash
[Detected format command: prettier --write ., black ., rustfmt, gofmt -w ., etc.]
` ``

### Build
` ``bash
[Detected build command: npm run build, python -m build, cargo build --release, go build, etc.]
` ``

## Code Style

- Follow [detected formatter] configuration
- [Language-specific conventions: PEP 8, Google Style Guide, Effective Go, etc.]
- [Max line length from formatter config]
- [Import ordering rules if detected]

## Testing Conventions

- Test framework: [detected framework]
- Test files: [pattern like *_test.go, test_*.py, *.test.ts]
- Coverage target: [if detected in config, else "aim for >80%"]
- [Framework-specific patterns: describe/it, test classes, table tests, etc.]

## Git Workflow

- Branch naming: `feature/`, `fix/`, `docs/`
- Commit messages: Conventional Commits format
- Pre-commit hooks: [if detected: lint-staged, husky, pre-commit, etc.]

## Context Management
- Use `/compact` when context gets large (check statusline)
- Use `/clear` between unrelated tasks
- Use `@import` to reference detailed docs instead of inlining them
- Keep CLAUDE.md under 500 lines — split into `@imports` if growing

## Verification
- Always run tests after changes: `[detected test command]`
- Run linter before committing: `[detected lint command]`
- Build must pass: `[detected build command]`

## Key Conventions

[Language/framework-specific conventions:]
[For React: component structure, hooks usage, prop types]
[For Python: type hints, docstrings format]
[For Go: error handling, package naming]
[For Rust: Result/Option handling, trait bounds]

## Skill Lifecycle

After completing a major task, evaluate:
- **Create a new skill** if the task involved a repeatable workflow that would benefit from a dedicated `/skill-name` command
- **Update an existing skill** if you used a skill and found it missing steps, producing suboptimal results, or not covering edge cases you encountered

When creating or updating skills, follow the format in `.claude/skills/` and keep them focused on a single workflow.

---

**For detailed guides, see:**
- Skills: `.claude/skills/`
- Custom agents: `.claude/agents/`
- Rules: `.claude/rules/`
```
