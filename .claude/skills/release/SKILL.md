# /release — Bump Version and Tag a Release

Automates semantic version bumps, changelog updates, and git tagging.

## Usage

```
/release <major|minor|patch>
```

## Steps

1. **Read current version** from `VERSION` file at repo root
2. **Bump version** according to the argument:
   - `major`: X.0.0 (breaking changes)
   - `minor`: X.Y+1.0 (new features)
   - `patch`: X.Y.Z+1 (fixes)
3. **Update `CHANGELOG.md`**:
   - Rename `## [Unreleased]` section to `## [NEW_VERSION] - YYYY-MM-DD`
   - Add a fresh empty `## [Unreleased]` section above it
   - Update comparison links at the bottom of the file
4. **Update `VERSION`** file with the new version string
5. **Commit** all changes with message: `chore: release vNEW_VERSION`
6. **Create git tag** `vNEW_VERSION` on the new commit

## Rules

- Abort if the `## [Unreleased]` section in CHANGELOG.md is empty (nothing to release)
- Abort if working tree is dirty (uncommitted changes besides the release edits)
- Always verify the final state: `cat VERSION && git tag -l 'vNEW_VERSION'`
- Do NOT push — let the user decide when to push

## Example

```
/release patch
# VERSION: 0.1.0 → 0.1.1
# CHANGELOG: [Unreleased] → [0.1.1] - 2026-02-09
# Commit: chore: release v0.1.1
# Tag: v0.1.1
```
