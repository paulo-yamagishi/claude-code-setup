# Plugin System

## What Are Plugins?

Plugins are structured extensions that add capabilities to Claude Code. A plugin bundles skills, agents, hooks, MCP servers, and LSP servers into a single distributable package.

## Plugin Structure

```
my-plugin/
  .claude-plugin/
    plugin.json          # Manifest (required)
  commands/              # Skills as Markdown
  agents/                # Custom agent definitions
  skills/                # Agent Skills with SKILL.md
  hooks/
    hooks.json           # Event handlers
  .mcp.json              # MCP server configs
  .lsp.json              # LSP server configs
```

### Component Details

| Directory/File | Purpose |
|---|---|
| `.claude-plugin/plugin.json` | Required manifest with name, description, version, author |
| `commands/` | User-invocable skills as Markdown files |
| `agents/` | Custom agent definitions (same format as `.claude/agents/`) |
| `skills/` | Agent skills with `SKILL.md` files |
| `hooks/hooks.json` | Event handlers (same format as settings hooks, scoped to plugin) |
| `.mcp.json` | MCP server configurations loaded when plugin is enabled |
| `.lsp.json` | LSP server configurations for code intelligence |

## Plugin Manifest (plugin.json)

The `plugin.json` manifest is required and defines the plugin metadata:

```json
{
  "name": "my-plugin",
  "description": "A plugin that does useful things",
  "version": "1.0.0",
  "author": "Your Name",
  "homepage": "https://github.com/you/my-plugin",
  "repository": "https://github.com/you/my-plugin",
  "license": "MIT"
}
```

| Field | Description |
|-------|-------------|
| `name` | Plugin identifier (used for namespacing) |
| `description` | What the plugin does |
| `version` | Semantic version |
| `author` | Plugin author |
| `homepage` | Plugin homepage URL |
| `repository` | Source repository URL |
| `license` | License identifier |

## Plugin Namespacing

Plugin skills are namespaced using the `plugin-name:skill-name` format to avoid conflicts with other plugins or project-level skills:

```
/my-plugin:fix-issue 42
/linter-plugin:check-style
```

Skills are defined as `SKILL.md` files inside the `skills/` or `commands/` directory of the plugin.

## Plugin Hooks

Plugin hooks are defined in `hooks/hooks.json`. The format is the same as settings hooks but scoped to the plugin. Use the `${CLAUDE_PLUGIN_ROOT}` variable for paths relative to the plugin directory:

```json
{
  "PostToolUse": [
    {
      "matcher": "Write",
      "hooks": [
        {
          "type": "command",
          "command": "${CLAUDE_PLUGIN_ROOT}/scripts/format.sh \"$CLAUDE_TOOL_INPUT_PATH\""
        }
      ]
    }
  ]
}
```

## LSP Servers (.lsp.json)

The `.lsp.json` file configures Language Server Protocol servers that provide code intelligence features like diagnostics, completions, and hover information:

```json
{
  "servers": [
    {
      "command": "typescript-language-server",
      "args": ["--stdio"],
      "extensionToLanguage": {
        ".ts": "typescript",
        ".tsx": "typescriptreact",
        ".js": "javascript",
        ".jsx": "javascriptreact"
      }
    }
  ]
}
```

The `extensionToLanguage` mapping tells Claude which file extensions correspond to which language identifiers, enabling the LSP server to provide appropriate intelligence for each file type.

## Discovering and Installing Plugins

### Browsing Plugins

Use the `/plugin` command to browse available plugins from configured marketplaces:

```
/plugin
```

### Installing a Plugin

Install a plugin from a marketplace:

```
/plugin install my-plugin
```

### Testing Local Plugins

Load a plugin from a local directory for development and testing:

```bash
claude --plugin-dir ./my-plugin
```

### Enabling Plugins in Settings

Enable installed plugins in settings with the `enabledPlugins` field:

```json
{
  "enabledPlugins": {
    "my-plugin": true
  }
}
```

## Plugin Marketplaces

Plugins can be distributed from multiple source types:

| Source | Description |
|--------|-------------|
| `github` | GitHub repository |
| `git` | Any git repository |
| `url` | Direct URL |
| `npm` | npm package |
| `file` | Local file path |
| `directory` | Local directory path |
| `hostPattern` | Pattern-based host matching |

Create and distribute plugin collections by publishing them to any of these sources.

## Converting Standalone Config to a Plugin

If you have project-level configuration in `.claude/` that you want to reuse across projects, convert it to a plugin:

1. Create the plugin directory with `.claude-plugin/plugin.json`
2. Move skills from `.claude/commands/` to `commands/`
3. Move agents from `.claude/agents/` to `agents/`
4. Move MCP config from `.claude/.mcp.json` to `.mcp.json`
5. Extract reusable hooks from settings into `hooks/hooks.json`
6. Add any LSP configs to `.lsp.json`
7. Test with `claude --plugin-dir ./my-plugin`

## Security Considerations

**Plugins run with your user permissions.** A plugin's hooks, MCP servers, and agents can access anything your user account can access.

- **Audit plugins before use.** Review the plugin source, especially hooks and MCP server configurations.
- **Limit scope.** Only enable plugins you actively need.
- **Do not store secrets in plugin configs committed to version control.** Use environment variables.
- **Be cautious with community plugins.** Prefer plugins from known authors with visible source code.

## Best Practices

- **Install only what you need.** Each plugin adds startup time and token overhead.
- **Use project-level enablement for project-specific plugins** and user-level for personal tools.
- **Keep plugins updated.** Plugin implementations improve over time with bug fixes and new features.
- **Document plugin dependencies.** If your project requires specific plugins, note them in your CLAUDE.md so team members know to install them.
- **Test plugin interactions.** After adding a plugin, verify Claude can successfully use its tools before relying on it for important work.
