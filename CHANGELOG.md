# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

## [1.0.0] - 2026-02-09

### Added

- `/evolve` skill — auto-maintains Claude Code configuration as the codebase evolves
- `/evolve` frontmatter with scoped `allowed-tools`
- User consent flow for `/evolve` Stop hook installation
- Biome detection in `/setup` and `/bootstrap` skills
- ESLint flat config (`eslint.config.*`) detection in `/bootstrap`
- Default deny rules in `/bootstrap` settings template
- Cross-platform notification hooks (macOS + Linux) in `/bootstrap`
- Output styles phase in `/bootstrap`
- Sandbox configuration phase in `/bootstrap`
- Model configuration phase in `/bootstrap`
- Agent team awareness in `/setup` audit and `/bootstrap` configuration
- Context management and verification sections in `/bootstrap` CLAUDE.md template
- `/init` acknowledgment in `/setup` skill
- Error recovery guidance in `/bootstrap`
- Async hooks documentation in core hooks guide
- Go and Rust project guides
- Bootstrap template extraction (`templates/bootstrap/`)
- Complete README rewrite with lifecycle narrative, detection matrix, FAQ, and badges

### Fixed

- CHANGELOG compare URLs now use correct repository name
- MCP GitHub env var consistency (`GITHUB_TOKEN` mapping)
- Bootstrap `allowed-tools` scoped to specific commands (was `Bash(*)`)
- Template settings, agents, and skills now match `/bootstrap` output
- Claude Code install method updated from deprecated `npm install -g` to native installer

### Changed

- `/evolve` asks for user consent before installing Stop hook
- Bootstrap skill restructured with extracted templates (under 400 lines)
- README completely rewritten for 1.0 release

## [0.1.0] - 2026-02-09

### Added

- `/setup` skill — audits Claude Code configuration and suggests improvements
- `/bootstrap` skill — auto-configures Claude Code for new projects
- `/verify-docs` skill — validates JSON, file paths, and settings keys
- `/sync-skills` skill — checks skill consistency with core guides
- `/research` skill — fetches official docs and community sources
- `/add-core-guide` and `/add-project-guide` skills
- `/update-readme` and `/refactor-docs` skills
- 26 core reference guides covering hooks, MCP, settings, agents, and more
- 4 language-specific project guides (Python, TypeScript, Go, Rust)
- Copy-paste ready config templates
- Prompt patterns and examples
- Context-specific rules via `.claude/rules/`

[Unreleased]: https://github.com/paulo-yamagishi/claude-code-setup/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/paulo-yamagishi/claude-code-setup/compare/v0.1.0...v1.0.0
[0.1.0]: https://github.com/paulo-yamagishi/claude-code-setup/releases/tag/v0.1.0
