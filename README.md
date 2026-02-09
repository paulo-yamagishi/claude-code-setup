# Claude Code Setup

Two skills for configuring Claude Code: `/setup` audits existing projects, `/bootstrap` scaffolds new ones.

Claude Code has 11+ configuration surfaces — CLAUDE.md, settings, hooks, skills, agents, MCP, rules, plugins, sandboxing, model config, and output styles. Most projects only use 2-3. These skills find what you're missing and help you set it up.

## Requirements

- [Claude Code](https://code.claude.com/docs) installed (`npm install -g @anthropic-ai/claude-code`)

## Install

### /setup (audit existing projects)

```bash
mkdir -p ~/.claude/skills/setup
curl -o ~/.claude/skills/setup/SKILL.md \
  https://raw.githubusercontent.com/paulo-yamagishi/claude-code-setup/main/skill/SKILL.md
```

### /bootstrap (configure new or existing projects)

```bash
mkdir -p ~/.claude/skills/bootstrap
curl -o ~/.claude/skills/bootstrap/SKILL.md \
  https://raw.githubusercontent.com/paulo-yamagishi/claude-code-setup/main/.claude/skills/bootstrap/SKILL.md
```

---

## /setup — Audit Your Config

Type `/setup` in any project and Claude will:

1. **Audit** your current config across 11 categories
2. **Suggest** improvements tailored to your tech stack — individual configs plus complete workflows (e.g., GitHub Issue → PR, TDD Loop, Deploy Pipeline)
3. **Apply** only what you approve — progressive disclosure shows a scorecard first, details on request

### Usage

| Command | What it does |
|---------|-------------|
| `/setup` | Full audit across all 11 categories |
| `/setup quick` | Show only gaps and missing items |
| `/setup hooks` | Audit only hooks configuration |
| `/setup mcp` | Audit only MCP server setup |
| `/setup settings` | Audit only settings and permissions |
| `/setup skills` | Audit only skills |
| `/setup agents` | Audit only custom agents |
| `/setup claude-md` | Audit only CLAUDE.md quality |
| `/setup rules` | Audit only rules |
| `/setup plugins` | Audit only plugins |
| `/setup sandbox` | Audit only sandboxing |
| `/setup model` | Audit only model configuration |
| `/setup styles` | Audit only output styles |

### What Gets Audited

| Category | What's checked |
|----------|---------------|
| CLAUDE.md | Exists, under 500 lines, has key sections |
| Settings | Permissions configured, deny rules, local overrides |
| Hooks | Auto-format, notifications, dangerous command blocking |
| Skills | Exist, proper format, common skills (fix-issue, code-review) |
| Agents | Exist, tool restrictions, appropriate model selection |
| MCP | Configured, no hardcoded secrets, minimal server count |
| Rules | Exist, split by topic, focused files |
| Plugins | Installed, code intelligence for typed languages, marketplace |
| Sandboxing | Enabled, auto-allow mode, allowed domains |
| Model Config | Model alias set, opusplan usage, effort level |
| Output Styles | Custom styles directory, team styles, settings |

### Example Output

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

## Top Suggestions

1. [Hooks] Add auto-format hook for Prettier (detected)
2. [Settings] Add deny rules for destructive commands
3. [Skills] Create fix-issue skill for GitHub workflow
4. [Plugins] Install TypeScript code intelligence plugin
5. [Sandbox] Enable sandbox for autonomous work

Say 'details 1' for config snippet, or 'bootstrap' to apply all suggestions automatically.
```

---

## /setup → /bootstrap Flow

Run `/setup` first, then say "bootstrap" in the same conversation. `/bootstrap` will pick up your audit results automatically — no re-detection needed.

```
/setup  ──audit──▶  scorecard + suggestions + handoff block
                         │
  user says "bootstrap"  │
                         ▼
/bootstrap  ──▶  "Found your /setup audit — focusing on the gaps."
                 • Prioritizes IMPROVE and MISSING categories
                 • Skips prompting for GOOD categories
                 • Uses suggestions as a checklist
```

Running `/bootstrap` standalone (without a prior `/setup`) works fine — it does full detection on its own.

---

## /bootstrap — Set Up a Project from Scratch

Type `/bootstrap` in any project — new or existing — and Claude will interactively configure everything for you.

### What It Does

`/bootstrap` runs a 9-phase interactive flow (Phase 0 + Phases 1–8):

0. **Check for setup report** — if `/setup` was run earlier in the conversation, uses its findings to focus work
1. **Detect** your project stack (language, framework, package manager, tools)
2. **Greenfield scaffolding** — if the directory is empty, asks what you want to build and initializes the project (installs dependencies, creates directory structure, starter files)
3. **Generate CLAUDE.md** — tailored to your stack with build commands, code style, and project conventions
4. **Configure settings** — permissions, deny rules, hooks (auto-format, lint-on-save, dangerous command blocking)
5. **Create skills** — `/fix-issue` and `/code-review` skills, plus conditional ones based on your stack
6. **Create agents** — code-reviewer, test-writer, and conditional agents (security-auditor for web apps, etc.)
7. **Set up MCP** — GitHub, database, Slack, and other servers based on what's detected
8. **Generate rules** — `.claude/rules/` files for code style, testing conventions, and git workflow

### Greenfield Support

Starting a brand new project? `/bootstrap` handles it:

- Asks your preferred language, framework, package manager, and project type
- Runs the appropriate init command (`npm init`, `cargo init`, `uv init`, `go mod init`, etc.)
- Installs framework + dev tools (formatter, linter, test runner)
- Creates standard directory structure (`src/`, `tests/`, config files)
- Then proceeds with full Claude Code configuration

### Usage

```
/bootstrap              # Full interactive setup
/bootstrap express      # Hint: Express.js project
/bootstrap fastapi      # Hint: FastAPI project
```

---

## Also In This Repo

| Path | Description |
|------|-------------|
| `skill/SKILL.md` | The `/setup` skill source |
| `.claude/skills/bootstrap/SKILL.md` | The `/bootstrap` skill source |
| `core/` | Reference guides for hooks, skills, MCP, settings, agents, model config, sandboxing, and more |
| `templates/` | Copy-paste templates for manual config |
| `projects/` | Language-specific guides (Python, TypeScript, etc.) |

## Contributing

Issues and PRs are welcome. If you find inaccurate config examples or missing Claude Code features, please open an issue.

## License

[MIT](LICENSE)

## Official Docs

- [Claude Code](https://code.claude.com/docs)
