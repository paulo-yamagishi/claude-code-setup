# Skill Examples

For skill format, frontmatter fields, and best practices, see [skills-guide.md](skills-guide.md).

---

## Real-World Skill Examples

### Tech Debt Scanner

Run at the end of every session to catch accumulated debt:

```markdown
---
name: techdebt
description: Scan codebase for duplicated code and tech debt patterns
allowed-tools:
  - Grep
  - Glob
  - Read
---

# Tech Debt Scan

Scan the codebase for:
1. Duplicated code blocks (3+ similar implementations)
2. TODO/FIXME/HACK comments added in the last week
3. Functions over 100 lines
4. Files with no test coverage

Report findings ranked by severity. Suggest concrete refactoring steps for the top 3 issues.
```

### Context Sync

Aggregate context from multiple tools into one dump before starting work:

```markdown
---
name: context-sync
description: Pull context from Slack, GitHub, and project docs
allowed-tools:
  - Bash(gh *)
  - Read
  - Glob
---

# Context Sync

Gather current project context:
1. Recent GitHub issues and PRs: `gh issue list --limit 10` and `gh pr list --limit 10`
2. Read the project CHANGELOG and recent commit messages
3. Summarize: what's in progress, what's blocked, what shipped recently

Present a brief status report I can scan in 30 seconds.
```

This pairs well with MCP servers for Slack, Google Drive, or Asana — see `core/mcp-guide.md` for setup.

### Analytics Engineer

For data teams using dbt or similar tools:

```markdown
---
name: analytics
description: Write and test dbt models
argument-hint: "[model-name]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash(dbt *)
  - Glob
---

# Analytics Engineer

Write or update the dbt model for $ARGUMENTS.
1. Read existing models in `models/` for conventions.
2. Write the model SQL following project patterns.
3. Run `dbt compile` to validate.
4. Run `dbt test` on the new model.
5. Update the schema YAML if adding new columns.
```

### Code Review

```markdown
---
name: code-review
description: Review staged changes for quality issues
allowed-tools:
  - Bash(git diff *)
  - Read
  - Glob
  - Grep
---

# Code Review

Review all staged changes (`git diff --cached`) for:
- Logic errors and bugs
- Security vulnerabilities
- Performance problems
- Code style violations
- Missing error handling

Provide a summary with severity ratings (critical, warning, info).
```

### Create Component

```markdown
---
name: create-component
description: Create a new React component with tests
argument-hint: "[ComponentName]"
allowed-tools:
  - Write
  - Read
  - Glob
---

# Create Component

Create a new React component called $ARGUMENTS.

1. Create `src/components/$ARGUMENTS/$ARGUMENTS.tsx`.
2. Create `src/components/$ARGUMENTS/$ARGUMENTS.test.tsx`.
3. Create `src/components/$ARGUMENTS/index.ts` with a re-export.
4. Follow existing component patterns in the codebase.
```

### Deploy

```markdown
---
name: deploy
description: Deploy the application to a specified environment
argument-hint: "[environment]"
allowed-tools:
  - Bash(./scripts/deploy.sh *)
  - Bash(npm run build)
  - Bash(npm test)
---

# Deploy

Deploy the application to $0.

1. Run the test suite.
2. Build the application for the target environment.
3. Run the deploy script: `./scripts/deploy.sh $0`.
4. Verify the deployment by checking the health endpoint.
5. Report the deployment status.
```
