---
description: Run pre-commit on all files or modified files
---

Run pre-commit checks on the repository.

**Options:**

- Run on all files: `uv run pre-commit run --all-files` (or `pre-commit run --all-files`)
- Run on staged files only: `uv run pre-commit run` (or `pre-commit run`)

Please execute the appropriate pre-commit command based on the current context. If there are any failures, show them and fix the issues before proceeding.

**Important:** Pre-commit is MANDATORY before committing - never skip it.
