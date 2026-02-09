---
name: fix-issue
description: Fix a GitHub issue by number — reads the issue, explores the code, and implements a fix
args:
  - name: issue_number
    description: The GitHub issue number to fix
    required: true
---

# Fix Issue

Fix GitHub issue #{{issue_number}}.

## Steps
1. Read the issue details: `gh issue view {{issue_number}}`
2. Understand the problem described in the issue
3. Explore the relevant code to find the root cause
4. Plan the fix
5. Implement the fix
6. Write or update tests to cover the fix
7. Run the test suite to verify
8. Summarize what was changed and why
