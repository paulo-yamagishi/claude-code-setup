---
name: bootstrap
description: Auto-configure Claude Code for the current project (CLAUDE.md, settings, hooks, skills, agents, MCP, rules)
argument-hint: "[framework-hint]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash(which *)
  - Bash(command -v *)
  - Bash(git config *)
  - Bash(git init *)
  - Bash(npm init *)
  - Bash(npm install *)
  - Bash(yarn init *)
  - Bash(pnpm init *)
  - Bash(bun init *)
  - Bash(cargo init *)
  - Bash(go mod init *)
  - Bash(uv init *)
  - Bash(poetry init *)
  - Bash(bundle init *)
  - Bash(composer init *)
---

# Bootstrap — Auto-Configure Claude Code for Your Project

Work through each phase sequentially, asking the user for input only where marked.

---

## Phase 0: Check for Setup Report

Before running detection, check if a `/setup` audit report exists earlier in this conversation. Look for a fenced code block with the `setup-report` language tag.

**If found:**
- Parse the stack, tools, category statuses, and suggestions
- Tell the user: "Found your /setup audit — focusing on the gaps it identified."
- Still run Phase 1 detection as normal, but use the setup report to **enhance** subsequent phases:
  - **Prioritize** categories marked IMPROVE or MISSING — generate config for these first
  - **Skip prompting** for categories marked GOOD — they're already configured
  - **Use setup's suggestions** as a checklist — ensure each one is addressed in the generated config
  - In Phase 8 summary, note which setup suggestions were applied

**If not found:**
- Proceed normally — this is the default path for standalone `/bootstrap` usage

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
- `biome.json`, `biome.jsonc` → Biome (format: `biome format --write`)

**Linters:**
- `.eslintrc*` → ESLint
- `eslint.config.*` → ESLint (flat config)
- `biome.json`, `biome.jsonc` → Biome (lint: `biome check`)
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

When a greenfield project is detected, ASK the user sequentially:

1. **Language/Runtime:** Node.js/TypeScript, Python, Rust, Go, Java, Ruby, PHP, Other
2. **Framework** (based on language): Node→Next.js/Express/Fastify/Hono, Python→Django/Flask/FastAPI, Rust→Actix/Axum/Rocket, Go→Gin/Echo/Chi, Java→Spring Boot/Quarkus, Ruby→Rails/Sinatra, PHP→Laravel/Symfony, or None
3. **Package Manager** (based on language): Node→npm/yarn/pnpm/bun, Python→pip/poetry/uv, others use defaults
4. **Project Type:** API/backend, Web app, CLI, Library, Monorepo
5. **Extras** (multi-select): Testing, Linting, Formatting, CI/CD, Docker, Database

**Scaffold:** Initialize project with selected package manager, install framework + dev tools, create directory structure (`src/`, `tests/`, plus type-specific subdirs), create starter files (entry point, config, `.gitignore`, `README.md`), and install selected extras.

After scaffolding, proceed to **Phase 2** with the now-known stack.

---

## Phase 2: Generate CLAUDE.md

If `CLAUDE.md` exists, ASK: "CLAUDE.md exists. Overwrite (o), merge (m), or skip (s)?"

Read the template from `templates/bootstrap/claude-md.md` and customize all `[bracketed placeholders]` based on detected stack. Write the result to `CLAUDE.md`.

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
    "deny": [
      "Bash(rm -rf *)",
      "Bash(git push --force *)",
      "Bash(git push -f *)",
      "Bash(git reset --hard *)",
      "Bash(git checkout -- .)",
      "Bash(git clean -f *)"
    ]
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
            "command": "[platform-detected notification command]"
          }
        ]
      }
    ]
  }
}
```

**Stop notification** (detect platform):
- If `which osascript` succeeds (macOS): `osascript -e 'display notification "Claude finished" with title "Claude Code"'`
- If `which notify-send` succeeds (Linux): `notify-send "Claude Code" "Claude finished" 2>/dev/null || true`
- Fallback: `echo "Claude finished"`

**Hook types available:**
- `"type": "command"` — runs a shell command (shown above)
- `"type": "prompt"` — single-turn LLM evaluation (use for the `/evolve` Stop hook)
- `"type": "agent"` — multi-turn subagent verification (use for complex checks)
- Add `"async": true` to any hook to run it in background without blocking Claude

**Format command construction:**
- **Prettier:** `prettier --write $CLAUDE_TOOL_INPUT_PATH`
- **Black:** `black $CLAUDE_TOOL_INPUT_PATH`
- **rustfmt:** `rustfmt $CLAUDE_TOOL_INPUT_PATH`
- **gofmt:** `gofmt -w $CLAUDE_TOOL_INPUT_PATH`
- **PHP CS Fixer:** `php-cs-fixer fix $CLAUDE_TOOL_INPUT_PATH`

### 3.2 Create .claude/settings.local.json Template

Create with empty `permissions` (allow/deny arrays) and `hooks` object. Add `.claude/settings.local.json` to `.gitignore`.

### 3.3 Sandbox Configuration

ASK: "Enable sandbox mode for reduced permission prompts? (y/n)"

If yes, add to settings:
```json
{
  "sandbox": {
    "mode": "sandbox",
    "network": {
      "allowedDomains": ["github.com"]
    }
  }
}
```

Customize `allowedDomains` based on detected stack:
- Node.js: add `registry.npmjs.org`
- Python: add `pypi.org`
- Rust: add `crates.io`
- Go: add `proxy.golang.org`

### 3.4 Model Configuration

ASK: "Set default model? (opus for complex projects, sonnet for balanced, haiku for fast/cheap)"

Add to settings based on selection:
- **opus:** `"model": "opus"`, `"effortLevel": "high"`
- **sonnet:** `"model": "sonnet"`, `"effortLevel": "high"`
- **haiku:** `"model": "haiku"`, `"effortLevel": "medium"`
- **skip:** Don't add model config (uses system default)

---

## Phase 4: Create Skills

Create `.claude/skills/` directory. Read skill templates from `templates/bootstrap/skills.md` and create each skill file, customizing `[bracketed placeholders]` based on detected stack. Create 4.1 (fix-issue) and 4.2 (code-review) always; create 4.3 (create-component) for React/Vue, 4.4 (migrate) for Django/Rails/Prisma, 4.5 (deploy) if deploy scripts exist.

---

## Phase 5: Create Custom Agents

Create `.claude/agents/` directory. Read agent templates from `templates/bootstrap/agents.md` and create each agent file, customizing `[bracketed placeholders]` based on detected stack. Always create 5.1 (code-reviewer) and 5.2 (test-writer). ASK "Include security-auditor agent? (y/n)" for 5.3.

### 5.4 Agent Team Support (Optional)

ASK: "Configure agent team support? (useful for complex multi-step tasks) (y/n)"

If yes, add to `.claude/settings.json`:
```json
{
  "teammateMode": "auto"
}
```

Suggest team patterns based on project type:
- **TDD projects:** code-reviewer + test-writer team
- **Full-stack:** frontend-agent + backend-agent + test-writer team
- **API projects:** api-designer + test-writer + security-auditor team

---

## Phase 6: MCP Servers (Interactive)

ASK the user: "Which MCP servers do you want to configure?"

Present options (multi-select): GitHub, Filesystem, PostgreSQL (if detected), MySQL (if detected), Slack, Linear, Notion.

### 6.1 Create .mcp.json

Read MCP server configs from `templates/bootstrap/mcp.md`. Include only the servers the user selected, wrap in `{ "mcpServers": { ... } }`, and write to `.mcp.json`.

### 6.2 Document Environment Variables

Create `.claude/ENV.md` with required variables for each configured MCP server (see template in `templates/bootstrap/mcp.md`).

---

## Phase 7: Rules

Create `.claude/rules/` directory. Read rule descriptions from `templates/bootstrap/rules.md` and generate each rule file (7.1 code-style, 7.2 testing, 7.3 git) based on detected stack. Extract actual conventions from the project's existing code.

---

## Phase 7.5: Output Styles

Create `.claude/output-styles/` directory.

### 7.5.1 concise.md

```markdown
---
name: concise
description: Brief, code-focused output with no emoji
---

Be concise. Focus on code, not explanation. Skip pleasantries. No emoji.
When showing changes, use diffs or code blocks — not prose.
Keep responses under 20 lines unless the user asks for more detail.
```

Note to user: "Custom output styles added. Create more in `.claude/output-styles/` to match your preferred response format."

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
- **After each phase**, list files created so far in case of failure
- **If a phase fails**, report: "Bootstrap partially completed through Phase N. Files created: [list]. You can re-run `/bootstrap` — it will detect existing files and offer to merge/skip."
- **For existing files**, always offer: overwrite (o), merge (m), or skip (s)

### User Interaction Points

ASK at: CLAUDE.md overwrite (Phase 2), MCP server selection (Phase 6), security-auditor (Phase 5.3), additional custom skills (after Phase 4), and final customization ("Would you like to customize any of these files now?").

### Verification

After each phase, verify files were created successfully. If Write fails, report immediately. For language-specific sections, use actual examples from the project where possible.

---

**END OF BOOTSTRAP SEQUENCE**
