# settings.json Reference

This file documents the fields in `settings.json.template`.

## `permissions.allow`

An array of tool permission rules that Claude Code can use without prompting. Evaluation order: deny → ask → allow (first match wins). Each entry can be:

- A plain tool name (e.g., `"Read"`, `"Write"`, `"Edit"`, `"Glob"`, `"Grep"`)
- A `Bash(command)` pattern that whitelists specific shell commands (e.g., `"Bash(npm test)"`)
- A `Bash(command *)` pattern with a wildcard to allow any arguments (e.g., `"Bash(git *)"`)

<!-- CUSTOMIZE: Add or remove tools based on your project's needs. Be restrictive — only allow commands you trust to run without review. -->

## `env`

Environment variables that Claude Code will set when running commands.

<!-- CUSTOMIZE: Add any environment variables your project needs (e.g., NODE_ENV, DATABASE_URL for local dev, PATH additions). Never put real secrets here — use a .env file or secret manager instead. -->
