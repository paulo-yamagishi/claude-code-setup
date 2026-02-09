---
name: add-core-guide
description: Create a new core/ reference guide following repo conventions
argument-hint: "[topic-name]"
allowed-tools:
  - Read
  - Write
  - Glob
  - Grep
  - WebFetch
---

# Add Core Guide

Create a new `core/{topic}-guide.md` file for the topic specified in `$ARGUMENTS`.

## Steps

1. **Validate topic:** Confirm `core/$ARGUMENTS-guide.md` does not already exist. If it does, report and stop.

2. **Fetch official docs:** Use WebFetch to read `https://docs.anthropic.com/en/docs/claude-code` pages relevant to the topic for authoritative content.

3. **Study existing guides:** Read 2-3 files in `core/` to extract the consistent structure:
   - Title format: `# Topic Guide`
   - Section pattern: overview → key concepts → configuration → examples → troubleshooting → cross-references
   - Style: imperative, concrete examples, valid JSON in code blocks, language-agnostic
   - Length: under 300 lines

4. **Generate the guide:**
   - Follow the extracted structure exactly
   - Use only verified information from official docs
   - Include at least 2 concrete JSON or bash code block examples
   - All JSON must be valid — no comments, no trailing commas
   - All settings keys and hook events must match official documentation
   - Keep language-agnostic — no framework-specific advice

5. **Add cross-references:**
   - Link to related core guides using relative paths: `../core/{related}-guide.md`
   - Grep other guides for mentions of the new topic and note where a backlink would help

6. **Verify:** Re-read the generated file and confirm:
   - Valid JSON in all code blocks
   - Under 300 lines
   - No duplicate of existing content in other guides
