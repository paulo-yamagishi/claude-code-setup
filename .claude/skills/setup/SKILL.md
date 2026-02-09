---
name: setup
description: Audit Claude Code configuration and suggest project-tailored improvements
argument-hint: "[quick|claude-md|settings|hooks|skills|agents|mcp|rules|plugins|sandbox|model|styles]"
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash(which *)
  - Bash(command -v *)
  - Bash(git config *)
---

# Claude Code Setup Advisor

You are auditing the current project's Claude Code configuration. Follow these phases in order.

## 1. Routing

Parse `$ARGUMENTS` to determine scope:
- Empty or unrecognized → full audit (all 11 categories)
- `quick` → full audit but only show IMPROVE and MISSING items, skip GOOD
- Category name → audit only that category: `claude-md`, `settings`, `hooks`, `skills`, `agents`, `mcp`, `rules`, `plugins`, `sandbox`, `model`, `styles`

## 2. Detection

Silently scan the project. Do NOT dump raw file contents to the user.

**Project stack:** Read `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `pom.xml`, `Gemfile`, `composer.json` (whichever exists). Note: language, framework, package manager.

**Tools:** Check for formatters (`.prettierrc*`, `[tool.black]`, `.rustfmt.toml`), linters (`.eslintrc*`, `eslint.config.*`, `[tool.ruff]`, `.golangci.yml`), test runners (test script in package.json, `[tool.pytest]`, `*_test.go`), lockfiles.

**Existing Claude config:** Check for `.claude/` directory, `CLAUDE.md`, `.claude/settings.json`, `.claude/settings.local.json`, `.claude/skills/`, `.claude/agents/`, `.claude/rules/`, `.claude/output-styles/`, `.mcp.json`. Check settings for `enabledPlugins`, `sandbox`, `model`, `effortLevel`, `outputStyle`.

**Git:** Check for `.git/`, remote origin URL (`git config --get remote.origin.url`), contributor count (`git shortlog -sn --no-merges | wc -l`).

**Workflow triggers:** CI/CD (`.github/workflows/`, `.gitlab-ci.yml`, `Jenkinsfile`), deploy (`Dockerfile`, `docker-compose.yml`, `fly.toml`, `vercel.json`, `netlify.toml`), monorepo (`turbo.json`, `nx.json`, `pnpm-workspace.yaml`, `lerna.json`), API (route/endpoint patterns, `openapi.yaml`, `swagger.json`), data (`*.ipynb`, `dbt_project.yml`, airflow DAGs), docs (`docs/` + `mkdocs.yml`/`docusaurus.config.js`/`conf.py`), production indicators (auth/session deps, `.env.production`, Docker).

Present a brief project profile (language, framework, tools) before the scorecard.

## 3. Audit Checklist

Score each category: **GOOD**, **IMPROVE**, **MISSING**.

### CLAUDE.md
1. Exists at `./CLAUDE.md` or `./.claude/CLAUDE.md`?
2. Under 500 lines? Uses `@imports` for detailed docs?
3. Has key sections: Commands (test/lint/build), Architecture, Code Style?

### Settings
1. `.claude/settings.json` exists with `permissions.allow` for detected safe commands?
2. `permissions.deny` configured for dangerous operations?
3. `.claude/settings.local.json` exists and is in `.gitignore`?

### Hooks
1. PostToolUse hook for auto-formatting on Write/Edit using detected formatter?
2. Stop hook for completion notification?
3. PreToolUse hook blocking dangerous Bash commands?

### Skills
1. `.claude/skills/` directory exists with at least one skill?
2. Skills use proper SKILL.md format with frontmatter?
3. Common skills present: `fix-issue`, `code-review`?

### Agents
1. `.claude/agents/` directory exists with custom agents?
2. Agents have tool restrictions (not inheriting all tools)?
3. Model set appropriately (sonnet for fast tasks, opus for complex)?

### MCP
1. `.mcp.json` exists with configured servers?
2. All secrets use `${VAR}` syntax, never hardcoded?
3. Only needed servers configured (each adds context overhead)?

### Rules
1. `.claude/rules/` directory exists with modular rule files?
2. Rules split by topic rather than one large file?
3. Rule files focused and under 40 lines each?

### Plugins
1. Any plugins installed (`enabledPlugins` in settings)?
2. For typed languages (TS, Rust, Go, Java): code intelligence plugin installed?
3. Plugin marketplace configured for team sharing?

### Sandboxing
1. Sandbox enabled (`sandbox` in settings or `/sandbox` used)?
2. Auto-allow mode configured for reduced permission prompts?
3. Allowed domains configured for network access?

### Model Config
1. `model` explicitly set in settings (not relying on default)?
2. Using `opusplan` for plan+execute workflow or appropriate alias?
3. `effortLevel` configured for cost/quality balance?

### Output Styles
1. `.claude/output-styles/` directory exists?
2. Custom styles defined for team consistency?
3. `outputStyle` set in settings if using non-default style?

## 4. Config Reference

Use these exact formats when generating suggestions.

### File Locations
- Settings: `.claude/settings.json` (project), `.claude/settings.local.json` (personal), `~/.claude/settings.json` (user)
- Managed: `/Library/Application Support/ClaudeCode/managed-settings.json` (macOS), `/etc/claude-code/managed-settings.json` (Linux)
- Skills: `.claude/skills/<name>/SKILL.md`
- Agents: `.claude/agents/<name>.md`
- Rules: `.claude/rules/*.md` (auto-imported, supports `paths` frontmatter)
- MCP: `.mcp.json` (project), `~/.claude.json` (user)
- CLAUDE.md: `./CLAUDE.md` or `./.claude/CLAUDE.md` (project), `~/.claude/CLAUDE.md` (user), `./CLAUDE.local.md` (personal)
- Output Styles: `.claude/output-styles/` (project), `~/.claude/output-styles/` (user)
- Plugins: `.claude-plugin/plugin.json` manifest, `commands/`, `agents/`, `skills/`, `hooks/hooks.json`, `.mcp.json`, `.lsp.json`

### Settings Schema
```
permissions.allow/deny/ask: ["Tool", "Tool(specifier)", "Tool(glob *)", "mcp__server__tool", "Task(AgentName)"]
permissions.defaultMode: default | acceptEdits | bypassPermissions | plan | delegate | dontAsk
Evaluation order: deny > ask > allow (first match wins)
Read/Edit rules: //abs, ~/home, /relative-to-settings, ./relative-to-cwd (gitignore patterns, * single dir, ** recursive)
model: default | sonnet | opus | haiku | opusplan | sonnet[1m] (or full model name)
effortLevel: low | medium | high
fastMode: true | false
outputStyle: style-name
sandbox: { mode, network: { allowedDomains, httpProxyPort }, excludedCommands, allowUnsandboxedCommands }
Other keys: env, hooks, enabledPlugins, additionalDirectories, teammateMode, respectGitignore, language
```

### Hook Events (all 14)
SessionStart, UserPromptSubmit, PreToolUse, PermissionRequest, PostToolUse, PostToolUseFailure, Notification, SubagentStart, SubagentStop, Stop, TeammateIdle, TaskCompleted, PreCompact, SessionEnd

### Hook Config Structure
```json
{ "EventName": [{ "matcher": "ToolName", "hooks": [{ "type": "command|prompt|agent", "command": "..." }] }] }
```
- Exit codes: 0=pass, 2=block (stderr fed to Claude), other=non-blocking
- Env vars: `CLAUDE_TOOL_NAME`, `CLAUDE_TOOL_INPUT`, `CLAUDE_TOOL_INPUT_PATH`, `CLAUDE_TOOL_OUTPUT`, `CLAUDE_SESSION_ID`, `CLAUDE_PROJECT_DIR`

### Frontmatter Quick Reference
- **SKILL.md**: name, description, argument-hint, allowed-tools, model, context (fork), agent, hooks, disable-model-invocation, user-invocable
- **Agent**: name (req), description (req), tools, disallowedTools, model, permissionMode, maxTurns, skills, mcpServers, hooks, memory (user/project/local)
- **Output Style**: name, description, keep-coding-instructions (default false)

### MCP Config
```json
{ "mcpServers": { "name": { "type": "stdio|sse|http", "command": "...", "args": [...], "env": { "KEY": "${VAR}" } } } }
```
Common servers: GitHub (`npx -y @modelcontextprotocol/server-github`), Filesystem, PostgreSQL, Slack.

## 5. Report — Progressive Disclosure

### Scoring System

Calculate a weighted 0–10 grade. Weights (total=10): CLAUDE.md/Settings/Hooks: 1.5 each (HIGH), Skills/Rules: 1.0 each (MEDIUM), Agents/MCP/Plugins/Sandboxing/Model/Styles: 0.5 each (LOW). Per category: GOOD=100%, IMPROVE=50%, MISSING=0% of weight. Grade labels:
- 9–10: "Excellent — power-user setup"
- 7–8: "Good — solid foundation, room to optimize"
- 5–6: "Fair — core pieces in place, key gaps remain"
- 3–4: "Basic — significant gaps affecting productivity"
- 0–2: "Fresh start — high-impact improvements available"

Example: 2 GOOD HIGH (1.5×2=3.0) + 1 IMPROVE HIGH (1.5×0.5=0.75) + 2 GOOD MEDIUM (1.0×2=2.0) + 6 MISSING LOW (0.5×0=0) = 5.75 → "Fair"
Calculate the exact score BEFORE writing the banner. Never revise mid-output.

### Output Template

Before generating any output, read `OUTPUT-FORMAT.md` from this skill's directory. It contains the exact template for health banner, scorecard, suggestions, and interactive prompt. Reproduce that format exactly — all 11 scorecard rows, emoji status indicators, box-drawing borders, impact-sorted suggestions with "Why" lines.

### Layer 3: Details (on request only)

When the user asks for details, show the full config snippet for that suggestion.

When the user wants to apply changes, tell them to run `/bootstrap` — this skill is read-only and cannot write files. `/bootstrap` has full write access and will generate the configuration this audit recommends.

## 6. Tailored Suggestion Examples

Use detected stack to generate project-specific suggestions:

- **Hooks**: Detected Prettier → `prettier --write "$CLAUDE_TOOL_INPUT_PATH"`. Detected Black → `black "$CLAUDE_TOOL_INPUT_PATH" 2>/dev/null || true`. Always suggest Stop notification and PreToolUse dangerous command blocker.
- **Skills**: Always fix-issue + code-review. React/Vue → create-component. Django/Rails/Prisma → migrate.
- **Agents**: Always code-reviewer (sonnet) + test-writer (sonnet). Production app → security-auditor (opus).
- **MCP**: GitHub remote → GitHub server. PostgreSQL/MySQL in deps → database server.
- **Settings**: Detected test/lint/build → permissions.allow. Standard deny rules for rm -rf, DROP, --force.
- **Plugins**: Typed language → code intelligence plugin. Team project → suggest creating a plugin marketplace.
- **Sandbox**: Any project → suggest enabling for reduced prompts. CI/CD → suggest auto-allow mode.
- **Model**: Complex project → suggest opusplan. Cost-sensitive → suggest effortLevel: medium.

## 7. Workflow Suggestions

Match detected project characteristics to workflows. Present 1-3 most relevant, numbered after individual suggestions:

- **GitHub Issue → PR**: GitHub remote + `gh` → `/fix-issue` skill + GitHub MCP + auto-format hook
- **TDD Loop**: Test framework → `/test-first` skill + test-writer agent + test-on-save hook
- **Database Migration**: Django/Rails/Prisma → `/migrate` skill + DB MCP + safety deny rules
- **Component Factory**: React/Vue/Svelte → `/create-component` skill
- **Code Review**: Multiple contributors → `/code-review` skill + reviewer agent (sonnet)
- **CI/CD**: `.github/workflows/` → headless mode tips + `--allowedTools` patterns
- **Deploy**: Dockerfile/fly.toml/vercel.json → `/deploy` skill + pre-deploy test hook
- **Security-First**: Auth deps + Docker → security-auditor agent (opus) + deny rules for secrets

Output in Layer 2: `6. [Workflow · MEDIUM] GitHub Issue → PR: end-to-end issue fixing`. Details on request (Layer 3).

## 8. Edge Cases

- **Fresh project (no .claude/):** Grade 0–2. Banner: "Fresh start". Focus on CLAUDE.md, auto-format hook, permissions.
- **Everything GOOD:** "Your setup looks solid!" Suggest power-user tweaks (agent teams, sandbox, opusplan).
- **Monorepo:** Scope detection to current working directory.
- **Existing config:** When showing details, ALWAYS display the full snippet. Direct users to `/bootstrap` to apply.
- **Minimize context:** Analyze internally. Only present scorecard + suggestions, not raw detection output.
