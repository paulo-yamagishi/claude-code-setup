# Claude Code Setup Advisor

A `/setup` skill that audits your Claude Code configuration and suggests project-tailored improvements.

## What It Does

Type `/setup` in any project and Claude will:

1. **Audit** your current config across 11 categories — CLAUDE.md, settings, hooks, skills, agents, MCP, rules, plugins, sandboxing, model config, output styles
2. **Suggest** improvements tailored to your tech stack — individual configs plus complete workflows (e.g., GitHub Issue → PR, TDD Loop, Deploy Pipeline)
3. **Apply** only what you approve — progressive disclosure shows a scorecard first, details on request

## Install

```bash
mkdir -p ~/.claude/skills/setup
curl -o ~/.claude/skills/setup/SKILL.md \
  https://raw.githubusercontent.com/YOUR_USER/claude-code-guide/main/skill/SKILL.md
```

Then type `/setup` in any Claude Code session.

## Usage

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

## What Gets Audited

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

## Example Output

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

Want details on any suggestion? Say 'details 1' or 'details all'.
To apply: 'apply 1, 3, 5' or 'apply all'.
```

## Also In This Repo

| Path | Description |
|------|-------------|
| `skill/SKILL.md` | The `/setup` skill source (install this) |
| `BOOTSTRAP.md` | Full greenfield setup automation (for new projects) |
| `core/` | Reference guides for hooks, skills, MCP, settings, agents, model config, sandboxing, and more |
| `templates/` | Copy-paste templates for manual config |
| `projects/` | Language-specific guides (Python, TypeScript, etc.) |

## Official Docs

- [Claude Code](https://code.claude.com/docs)
