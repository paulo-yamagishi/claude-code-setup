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
3. CLAUDE.md contains skill lifecycle instruction (evaluate creating/updating skills after major tasks)?

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

### Layer 1: Scorecard (always show)

```
## Setup Scorecard

| Category      | Status  | Notes                           |
|---------------|---------|----------------------------------|
| CLAUDE.md     | GOOD    | 120 lines, has key sections     |
| Settings      | IMPROVE | Missing deny rules              |
| Hooks         | MISSING | No hooks configured             |
| Skills        | MISSING | No skills directory             |
| Agents        | MISSING | No agents directory             |
| MCP           | GOOD    | GitHub configured               |
| Rules         | MISSING | No rules directory              |
| Plugins       | MISSING | No plugins installed            |
| Sandboxing    | MISSING | Not enabled                     |
| Model Config  | IMPROVE | Using default, no effort level  |
| Output Styles | MISSING | No custom styles                |

Score: 2/11 GOOD · 2 IMPROVE · 7 MISSING
```

### Layer 2: Suggestions (always show after scorecard)

For each IMPROVE or MISSING category, list **numbered, one-line** project-specific suggestions:

```
## Top Suggestions

1. [Hooks] Add auto-format hook for Prettier (detected)
2. [Settings] Add deny rules for destructive commands
3. [Skills] Create fix-issue skill for GitHub workflow
4. [Plugins] Install TypeScript code intelligence plugin
5. [Sandbox] Enable sandbox for autonomous work with less prompts
...
```

### Layer 3: Details (on request only)

End with: **"Want details on any suggestion? Say 'details 1' or 'details all'. To apply: 'apply 1, 3, 5' or 'apply all'."**

When the user asks for details, show the full config snippet for that suggestion. When they say apply, write the config files. The standard permission prompt serves as confirmation.

## 6. Tailored Suggestion Examples

Use detected stack to generate project-specific suggestions:

- **Hooks**: Detected Prettier → `prettier --write "$CLAUDE_TOOL_INPUT_PATH"`. Detected Black → `black "$CLAUDE_TOOL_INPUT_PATH" 2>/dev/null || true`. Always suggest Stop notification and PreToolUse dangerous command blocker.
- **Skills**: Always fix-issue + code-review. React/Vue → create-component. Django/Rails/Prisma → migrate. Always suggest skill lifecycle instruction in CLAUDE.md if missing.
- **Agents**: Always code-reviewer (sonnet) + test-writer (sonnet). Production app → security-auditor (opus).
- **MCP**: GitHub remote → GitHub server. PostgreSQL/MySQL in deps → database server.
- **Settings**: Detected test/lint/build → permissions.allow. Standard deny rules for rm -rf, DROP, --force.
- **Plugins**: Typed language → code intelligence plugin. Team project → suggest creating a plugin marketplace.
- **Sandbox**: Any project → suggest enabling for reduced prompts. CI/CD → suggest auto-allow mode.
- **Model**: Complex project → suggest opusplan. Cost-sensitive → suggest effortLevel: medium.

## 7. Workflow Suggestions

After individual suggestions, add a **Recommended Workflows** subsection. Match detected project characteristics to these workflows and present the 1-3 most relevant, numbered continuously after individual suggestions:

| Workflow | Trigger | Sets up |
|----------|---------|---------|
| GitHub Issue → PR | GitHub remote + `gh` CLI | `/fix-issue` skill + GitHub MCP + `Bash(gh *)` allow + auto-format hook |
| TDD Loop | Test framework detected | `/test-first` skill + test-writer agent + PostToolUse test-on-save hook |
| Database Migration | Django/Rails/Prisma/Drizzle in deps | `/migrate` skill + DB MCP + migration safety deny rules |
| Component Factory | React/Vue/Svelte in deps | `/create-component` skill following detected patterns |
| CI/CD Integration | `.github/workflows/` or `.gitlab-ci.yml` | Headless mode tips + `--allowedTools` patterns |
| Code Review Pipeline | Multiple contributors in git log | `/code-review` skill + code-reviewer agent (sonnet) + security-reviewer agent |
| Monorepo | `turbo.json`, `nx.json`, `pnpm-workspace.yaml`, `lerna.json` | Scoped CLAUDE.md per package + workspace-aware commands |
| API Development | Express/FastAPI/Django REST/NestJS in deps | API conventions rule + endpoint testing skill |
| Data Pipeline | pandas/dbt/airflow/spark in deps | `/analyze` skill + notebook-aware workflow + data validation hooks |
| Deploy Pipeline | Dockerfile, `fly.toml`, `vercel.json`, `netlify.toml` | `/deploy` skill + pre-deploy test hook |
| Documentation | `docs/` + mkdocs/docusaurus/sphinx config | `/docs` skill + doc-writer agent + link-checker hook |
| Security-First | Auth deps, `.env.production`, Docker | security-auditor agent (opus) + audit hook + deny rules for secrets |

Output in Layer 2 as one-liners: `6. [Workflow] GitHub Issue → PR: skill + MCP + hooks for end-to-end issue fixing`. When user says "details N", show the full workflow breakdown with all config snippets (Layer 3).

## 8. Edge Cases

- **Fresh project (no .claude/):** "Fresh project — here are the 3 highest-impact items." Focus on CLAUDE.md, auto-format hook, permissions.
- **Everything GOOD:** "Your setup looks solid!" Suggest power-user tweaks (agent teams, sandbox, opusplan).
- **Monorepo:** Scope detection to current working directory.
- **Ambiguous detection:** Ask the user to clarify.
- **Existing config:** When applying, ALWAYS show content first. NEVER silently overwrite.
- **Minimize context:** Analyze internally. Only present scorecard + suggestions, not raw detection output.
