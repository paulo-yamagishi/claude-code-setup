# Rule Templates for /bootstrap

## 7.1 .claude/rules/code-style.md

Generate based on detected stack. Include:
- Detected formatter and its config file path
- Language-specific naming conventions (camelCase/snake_case/PascalCase)
- Import ordering rules from linter config
- Max line length from formatter config
- Key idioms for the detected language (e.g., `const` over `let` for JS, type hints for Python, error handling for Go)

Keep under 40 lines. Extract actual conventions from the project's existing code.

## 7.2 .claude/rules/testing.md

Generate based on detected test framework. Include:
- Test runner command
- Test file naming pattern and location
- Arrange/Act/Assert structure example in the project's language
- Coverage target (from config or default 80%)
- Mocking approach used in the project

Keep under 30 lines.

## 7.3 .claude/rules/git.md

Generate standard git rules:
- Branch naming: `feature/`, `fix/`, `docs/`, `refactor/`
- Conventional Commits format: `type(scope): subject`
- Types: feat, fix, docs, style, refactor, test, chore
- Pre-commit hooks if detected, recommend if not
- PR conventions

Keep under 30 lines.
