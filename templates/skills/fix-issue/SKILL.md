---
name: fix-issue
description: Fix a GitHub issue by number — reads the issue, explores the code, and implements a fix
argument-hint: "<issue_number>"
allowed-tools:
  - Read
  - Edit
  - Write
  - Glob
  - Grep
  - Bash
---

# Fix Issue

Fix GitHub issue #$ARGUMENTS.

## Steps
1. Read the issue details: `gh issue view $ARGUMENTS`
2. Understand the problem described in the issue
3. Explore the relevant code to find the root cause
4. Plan the fix
5. Implement the fix
6. Write or update tests to cover the fix
7. Run the test suite to verify
8. Summarize what was changed and why
