# Git Workflow with Claude Code

## Commit Messages

Claude Code writes concise commit messages that focus on **why** a change was made, not what was changed. The diff already shows what changed; the message should explain the intent.

- Keep messages to 1-2 sentences.
- Use active voice: "Add user validation" not "User validation was added."
- Be specific: "Fix race condition in session cleanup" not "Fix bug."

### Co-Authored-By Trailer

Claude appends a co-authorship trailer to every commit it creates:

```
Fix rate limiting bypass in authentication middleware

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
```

### Style Matching

Before committing, Claude checks `git log` to read recent commit messages and match the repository's existing style. If your team uses conventional commits (`feat:`, `fix:`, `chore:`), Claude will follow that pattern.

## Branching

- **Feature branches:** Create descriptive branch names for new work (`feature/add-user-search`, `fix/session-timeout`).
- **Keep branches focused.** One branch per feature or fix.
- Claude can create and switch branches when asked, but will not do so unprompted.

## Pull Request Workflow

Claude uses the `gh` CLI for all GitHub operations. When asked to create a PR, Claude:

1. Checks `git status` and `git diff` to understand changes.
2. Reviews the full commit history for the branch.
3. Drafts a PR with a short title (under 70 characters) and a body containing:
   - A summary section with 1-3 bullet points.
   - A test plan with actionable checklist items.
4. Pushes the branch and creates the PR using `gh pr create`.

## What Claude Does Automatically

**Stages specific files by name.** Claude never uses `git add -A` or `git add .` because these can accidentally include sensitive files (`.env`, credentials) or large binaries. Instead, Claude adds only the files it changed.

**Checks `git log` for commit style.** Claude reads recent history to match your team's conventions.

**Checks `git status` and `git diff` before committing.** Claude reviews what will be committed to make sure it is correct.

## What Claude Never Does Without Asking

These actions are never taken unless you explicitly request them:

| Action | Risk |
|--------|------|
| `git push` | Publishes changes to the remote. |
| `git push --force` | Overwrites remote history. Claude will warn even if asked. |
| `git reset --hard` | Destroys uncommitted work permanently. |
| `git checkout .` / `git restore .` | Discards all local changes. |
| `git clean -f` | Deletes untracked files permanently. |
| `git branch -D` | Force-deletes a branch. |
| `git commit --amend` on published commits | Rewrites history others may have pulled. |
| Committing at all | Claude only commits when you ask it to. |

## Pre-Commit Hooks

If a pre-commit hook fails:

1. The commit did **not** happen. No changes were recorded.
2. Claude reads the hook output to understand the failure.
3. Claude fixes the issue (formatting, linting, etc.).
4. Claude re-stages the files.
5. Claude creates a **new** commit. It never uses `--amend` after a hook failure, because amending would modify the previous (unrelated) commit and risk destroying earlier work.

## Best Practices

**Commit frequently.** Small, focused commits are easier to review, revert, and understand.

**Keep commits focused.** Each commit should represent one logical change. Do not mix a bug fix with an unrelated refactor.

**Do not commit secrets.** Never commit `.env` files, API keys, credentials, or tokens. Claude will warn you if you ask it to commit files that likely contain secrets.

**Review before pushing.** After Claude creates commits, review them with `git log` and `git diff` before pushing to the remote.

**Use feature branches.** Do not commit directly to `main` or `master` for non-trivial changes.

**Write meaningful messages.** A good commit message saves time for everyone who reads the history later -- including future you.
