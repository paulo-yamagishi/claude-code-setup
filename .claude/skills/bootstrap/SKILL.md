---
name: bootstrap
description: Auto-configure Claude Code for the current project (CLAUDE.md, settings, hooks, skills, agents, MCP, rules)
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash(*)
---

# Bootstrap — Auto-Configure Claude Code for Your Project

Work through each phase sequentially, asking the user for input only where marked.

---

## Phase 1: Detect Project Stack

### 1.1 Read Project Metadata

Check for these files (in priority order):
- `package.json` (Node.js/JavaScript/TypeScript)
- `pyproject.toml` or `setup.py` (Python)
- `Cargo.toml` (Rust)
- `go.mod` (Go)
- `pom.xml` or `build.gradle` (Java/JVM)
- `Gemfile` (Ruby)
- `composer.json` (PHP)

### 1.2 Detect Tools

**Package managers:**
- Check for lockfiles: `package-lock.json` (npm), `yarn.lock`, `pnpm-lock.yaml`, `bun.lockb`, `poetry.lock`, `Cargo.lock`, `go.sum`, etc.

**Formatters:**
- `.prettierrc`, `.prettierrc.json`, `prettier.config.js` → Prettier
- `pyproject.toml` with `[tool.black]` → Black
- `.rustfmt.toml` → rustfmt
- `.gofmt` or standard Go project → gofmt
- `.php-cs-fixer.php` → PHP CS Fixer

**Linters:**
- `.eslintrc*` → ESLint
- `pyproject.toml` with `[tool.ruff]` or `.ruff.toml` → Ruff
- `pylint.rc` → Pylint
- `clippy.toml` → Clippy
- `.golangci.yml` → golangci-lint

**Test runners:**
- `package.json` scripts: `"test"` → npm test / jest / vitest
- `pytest.ini` or pytest in deps → pytest
- `Cargo.toml` → cargo test
- `*_test.go` files → go test
- `phpunit.xml` → PHPUnit

**Frameworks:**
- React: `react` in dependencies
- Vue: `vue` in dependencies
- Next.js: `next` in dependencies
- Django: `django` in dependencies
- Flask: `flask` in dependencies
- Rails: `Gemfile` with `rails`
- Express: `express` in dependencies

### 1.3 Map Directory Structure

Scan for common patterns:
- Source: `src/`, `lib/`, `app/`, `pkg/`
- Tests: `tests/`, `test/`, `__tests__/`, `*_test.go`, `*_test.py`
- Config: `config/`, `.config/`
- Docs: `docs/`, `documentation/`
- Build output: `dist/`, `build/`, `target/`, `.next/`

### 1.4 Check Existing Config

- `.claude/` directory exists?
- `CLAUDE.md` exists?
- `.mcp.json` exists?
- Git repository? Check for `.git/`

### 1.5 Greenfield Detection

If **no** project metadata files were found in 1.1 (no `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, etc.) **and** no source directories exist (`src/`, `lib/`, `app/`, `pkg/`), this is a **greenfield project**. Skip directly to **Phase 1.5** below.

Otherwise, **report findings to user:** "I've detected [language/framework]. Ready to configure?" and continue to Phase 2.

---

## Phase 1.5: Interactive Stack Selection (Greenfield Only)

When a greenfield project is detected, interactively ask the user to define their stack.

### Step 1: Language/Runtime

ASK: "What language/runtime will this project use?"

Options:
1. Node.js / TypeScript
2. Python
3. Rust
4. Go
5. Java
6. Ruby
7. PHP
8. Other (specify)

### Step 2: Framework

ASK based on language selection:

- **Node.js/TypeScript:** Next.js, Express, Fastify, Hono, None
- **Python:** Django, Flask, FastAPI, None
- **Rust:** Actix, Axum, Rocket, None (binary)
- **Go:** Gin, Echo, Chi, None (standard library)
- **Java:** Spring Boot, Quarkus, None
- **Ruby:** Rails, Sinatra, None
- **PHP:** Laravel, Symfony, None

### Step 3: Package Manager

ASK based on language:

- **Node.js:** npm, yarn, pnpm, bun
- **Python:** pip, poetry, uv
- **Rust:** cargo (default)
- **Go:** go modules (default)
- **Java:** Maven, Gradle
- **Ruby:** bundler (default)
- **PHP:** composer (default)

### Step 4: Project Type

ASK: "What type of project is this?"

Options:
1. API / backend service
2. Web application
3. CLI tool
4. Library / package
5. Monorepo

### Step 5: Extras

ASK (multi-select): "Which extras do you want?"

Options:
1. Testing framework
2. Linting
3. Formatting
4. CI/CD (GitHub Actions)
5. Docker
6. Database

### Step 6: Scaffold the Project

Based on the answers, execute the following:

**Initialize project** using the selected package manager:
- Node.js: `npm init -y` / `yarn init -y` / `pnpm init` / `bun init`
- Python: `uv init` / `poetry init --no-interaction` / create `pyproject.toml`
- Rust: `cargo init` · Go: `go mod init [module-name]` (ASK for module name)
- Java: `mvn archetype:generate` / `gradle init` · Ruby: `bundle init` · PHP: `composer init --no-interaction`

**Install framework + dev tools** (formatter, linter, test runner) appropriate for the language.

**Create directory structure:** `src/`, `tests/`, plus project-type-specific subdirectories (e.g., `src/routes/` for APIs, `src/components/` for web apps, `src/commands/` for CLIs).

**Create starter files:** language entry point (`src/index.ts`, `src/main.py`, etc.), config files (tsconfig.json, ruff in pyproject.toml, etc.), `.gitignore`, and `README.md`.

**If extras selected:** install testing framework + sample test, create `.github/workflows/ci.yml` for CI/CD, create `Dockerfile` + `.dockerignore` for Docker, install ORM/driver for database.

After scaffolding, proceed to **Phase 2** with the now-known stack.

---

## Phase 2: Generate CLAUDE.md

If `CLAUDE.md` exists, ASK: "CLAUDE.md exists. Overwrite (o), merge (m), or skip (s)?"

Create project-tailored instructions:

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

Write this to `CLAUDE.md`.

---

## Phase 3: Configure Settings

### 3.1 Create .claude/settings.json

Build permissions based on detected tools:

```json
{
  "permissions": {
    "allow": [
      "Read",
      "Write",
      "Edit",
      "Glob",
      "Grep",
      "Bash([detected test command])",
      "Bash([detected lint command])",
      "Bash([detected format command])",
      "Bash([detected build command])",
      "Bash(git status)",
      "Bash(git diff)",
      "Bash(git add *)",
      "Bash(git commit *)",
      "Bash(git push)",
      "Bash(git log *)"
    ],
    "deny": []
  },
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "[detected format command] \"$CLAUDE_TOOL_INPUT_PATH\""
          }
        ]
      },
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "[detected format command] \"$CLAUDE_TOOL_INPUT_PATH\""
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude finished\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

**Format command construction:**
- **Prettier:** `prettier --write $CLAUDE_TOOL_INPUT_PATH`
- **Black:** `black $CLAUDE_TOOL_INPUT_PATH`
- **rustfmt:** `rustfmt $CLAUDE_TOOL_INPUT_PATH`
- **gofmt:** `gofmt -w $CLAUDE_TOOL_INPUT_PATH`
- **PHP CS Fixer:** `php-cs-fixer fix $CLAUDE_TOOL_INPUT_PATH`

### 3.2 Create .claude/settings.local.json Template

```json
{
  "permissions": {
    "allow": [],
    "deny": []
  },
  "hooks": {}
}
```

Add to `.gitignore`: `.claude/settings.local.json`

---

## Phase 4: Create Skills

Create `.claude/skills/` directory with essential skills.

### 4.1 Always: .claude/skills/fix-issue/SKILL.md

```markdown
---
name: fix-issue
description: Fetch GitHub issue, implement fix, test, and commit
argument-hint: "[issue-number]"
allowed-tools:
  - Bash(gh issue view *)
  - Read
  - Write
  - Edit
  - Grep
  - Glob
---

# Fix GitHub Issue

1. Fetch issue details: `gh issue view $ARGUMENTS`
2. Analyze the issue and identify affected files
3. Implement the fix
4. Run tests: `[detected test command]`
5. If tests pass, commit: `git commit -m "fix: resolve #$ARGUMENTS"`
6. Ask user if they want to push
```

### 4.2 Always: .claude/skills/code-review/SKILL.md

```markdown
---
name: code-review
description: Review staged changes for quality and conventions
allowed-tools:
  - Bash(git diff *)
  - Read
  - Grep
  - Glob
---

# Code Review

1. Run `git diff --staged` to see changes
2. Review for: style violations, bugs, missing tests, security issues
3. Run linter: `[detected lint command]`
4. Provide feedback with severity: CRITICAL, WARN, SUGGESTION
```

### 4.3 Conditional: .claude/skills/create-component/SKILL.md (React/Vue)

```markdown
---
name: create-component
description: Create a new component with tests
argument-hint: "[ComponentName]"
allowed-tools:
  - Write
  - Read
  - Glob
---

# Create Component

1. Create `src/components/$ARGUMENTS/$ARGUMENTS.[ext]`
2. Create `src/components/$ARGUMENTS/$ARGUMENTS.test.[ext]`
3. Create `src/components/$ARGUMENTS/index.[ext]` with re-export
4. Follow existing component patterns in the codebase
5. Run tests: `[detected test command]`
```

### 4.4 Conditional: .claude/skills/migrate/SKILL.md (Django/Rails/Prisma)

```markdown
---
name: migrate
description: Create and apply database migration
argument-hint: "[migration-name]"
allowed-tools:
  - Bash
  - Read
  - Write
---

# Database Migration

1. Create migration: `[python manage.py makemigrations / rails g migration / prisma migrate dev]`
2. Review generated migration file
3. Apply migration
4. Update schema docs if present
```

### 4.5 Conditional: .claude/skills/deploy/SKILL.md (if deploy scripts exist)

```markdown
---
name: deploy
description: Deploy to specified environment
argument-hint: "[environment]"
allowed-tools:
  - Bash
  - Read
---

# Deploy

1. Verify clean working tree: `git status`
2. Run tests: `[test command]`
3. Build: `[build command]`
4. Deploy: `[detected deploy command]`
5. Verify deployment
```

---

## Phase 5: Create Custom Agents

Create `.claude/agents/` directory.

### 5.1 code-reviewer.md

```markdown
---
name: code-reviewer
description: Agent for reviewing code changes
tools:
  - Read
  - Bash
  - Grep
  - Glob
model: sonnet
permissionMode: acceptEdits
---

You are a code review agent. Your job:

1. Review code changes for:
   - Adherence to style guide (see CLAUDE.md)
   - Logical correctness
   - Performance issues
   - Security vulnerabilities
   - Test coverage

2. Provide structured feedback:
   - **CRITICAL:** Must fix before merge
   - **WARN:** Should fix
   - **SUGGESTION:** Nice to have

3. Never modify code. Only suggest changes.

4. Reference specific line numbers and files.

5. Praise good patterns when you see them.
```

### 5.2 test-writer.md

```markdown
---
name: test-writer
description: Writes tests for given files using [detected framework]
tools:
  - Read
  - Write
  - Edit
  - Bash
  - Grep
  - Glob
model: sonnet
---

You write tests using [detected framework: Jest/pytest/go test/etc.].

**Process:**

1. Read the source file provided
2. Identify all functions/methods/components
3. Write comprehensive tests:
   - Happy path
   - Edge cases
   - Error handling
   - [Framework-specific: mocking, fixtures, etc.]
4. Aim for >90% coverage
5. Run tests to verify: `[test command]`
6. Fix any failing tests

**Test file location:** [detected pattern: same dir, tests/ dir, __tests__, etc.]

**Naming:** [detected pattern: test_*.py, *.test.ts, *_test.go, etc.]
```

### 5.3 Conditional: security-auditor.md (for production apps)

ASK: "Include security-auditor agent? (y/n)"

```markdown
---
name: security-auditor
description: Audits code for security vulnerabilities
tools:
  - Read
  - Bash
  - Grep
  - Glob
model: opus
permissionMode: acceptEdits
---

You audit code for security issues:

1. **Input validation:** SQL injection, XSS, command injection
2. **Authentication/Authorization:** Broken access control, weak passwords
3. **Sensitive data:** Hardcoded secrets, exposed credentials
4. **Dependencies:** Known vulnerabilities (run: `[npm audit, pip-audit, cargo audit]`)
5. **Configuration:** Insecure defaults, debug mode in production
6. **Cryptography:** Weak algorithms, improper key management

Report findings with:
- **Severity:** CRITICAL / HIGH / MEDIUM / LOW
- **Location:** File and line number
- **Issue:** Clear description
- **Recommendation:** How to fix

Reference OWASP Top 10 and CWE standards.
```

---

## Phase 6: MCP Servers (Interactive)

ASK the user: "Which MCP servers do you want to configure?"

Present options (multi-select):
1. **GitHub** — PR/issue management (requires `gh` CLI)
2. **Filesystem** — Access outside project directory
3. **PostgreSQL** — Database queries (if detected)
4. **MySQL** — Database queries (if detected)
5. **Slack** — Team notifications
6. **Linear** — Issue tracking
7. **Notion** — Documentation

### 6.1 Create .mcp.json

Based on selections, build configuration:

Include only the servers the user selected. Use these configs:

**GitHub:**
```json
"github": {
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-github"],
  "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}" }
}
```

**Filesystem:**
```json
"filesystem": {
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/dir"]
}
```

**PostgreSQL:**
```json
"postgres": {
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-postgres"],
  "env": { "DATABASE_URL": "${DATABASE_URL}" }
}
```

**Slack:**
```json
"slack": {
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-slack"],
  "env": { "SLACK_BOT_TOKEN": "${SLACK_BOT_TOKEN}", "SLACK_TEAM_ID": "${SLACK_TEAM_ID}" }
}
```

Wrap selected servers in `{ "mcpServers": { ... } }` and write to `.mcp.json`.

### 6.2 Document Environment Variables

Create `.claude/ENV.md` with required variables:

```markdown
# Environment Variables for MCP Servers

Add these to your shell profile (`~/.zshrc` or `~/.bashrc`):

[For each configured MCP:]
export GITHUB_TOKEN="your_token_here"  # Get from: https://github.com/settings/tokens
export DATABASE_URL="postgresql://..."  # Your database connection string
[etc.]

Restart your terminal after adding these.
```

---

## Phase 7: Rules

Create `.claude/rules/` directory with modular rule files.

### 7.1 .claude/rules/code-style.md

Generate based on detected stack. Include:
- Detected formatter and its config file path
- Language-specific naming conventions (camelCase/snake_case/PascalCase)
- Import ordering rules from linter config
- Max line length from formatter config
- Key idioms for the detected language (e.g., `const` over `let` for JS, type hints for Python, error handling for Go)

Keep under 40 lines. Extract actual conventions from the project's existing code.

### 7.2 .claude/rules/testing.md

Generate based on detected test framework. Include:
- Test runner command
- Test file naming pattern and location
- Arrange/Act/Assert structure example in the project's language
- Coverage target (from config or default 80%)
- Mocking approach used in the project

Keep under 30 lines.

### 7.3 .claude/rules/git.md

Generate standard git rules:
- Branch naming: `feature/`, `fix/`, `docs/`, `refactor/`
- Conventional Commits format: `type(scope): subject`
- Types: feat, fix, docs, style, refactor, test, chore
- Pre-commit hooks if detected, recommend if not
- PR conventions

Keep under 30 lines.

---

## Phase 8: Summary and Next Steps

Print a summary listing:
1. All files created (with paths)
2. Detected stack (language, framework, package manager, test runner, formatter, linter)
3. Hooks enabled
4. MCP servers configured
5. Next steps: set env vars, test the setup, try `/fix-issue` or `/code-review`, commit config with `git add .claude/ CLAUDE.md .mcp.json`

---

## Additional Guidelines

### Error Handling

- If a file read fails, skip gracefully and note in summary
- If detection is ambiguous, ASK the user for clarification
- If tool commands can't be detected, use sensible defaults and note in CLAUDE.md

### User Interaction Points

- **Overwrite existing CLAUDE.md?** (Phase 2)
- **Which MCP servers?** (Phase 6)
- **Include security-auditor?** (Phase 5.3)
- **Any additional custom skills?** (After Phase 4)

### Verification Steps

After each phase, verify files were created successfully. If Write fails, report immediately.

### Customization Prompts

After completion, ASK: "Would you like to customize any of these files now?"

### Template Expansion

For language-specific sections, use actual examples from the project where possible (extract from existing code).

---

**END OF BOOTSTRAP SEQUENCE**
