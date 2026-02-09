# Skill Templates for /bootstrap

## 4.1 Always: .claude/skills/fix-issue/SKILL.md

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

## 4.2 Always: .claude/skills/code-review/SKILL.md

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

## 4.3 Conditional: .claude/skills/create-component/SKILL.md (React/Vue)

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

## 4.4 Conditional: .claude/skills/migrate/SKILL.md (Django/Rails/Prisma)

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

## 4.5 Conditional: .claude/skills/deploy/SKILL.md (if deploy scripts exist)

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
