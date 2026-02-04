---
name: precommit
description: Run pre-commit hooks on staged or all files. Use when checking code quality before commits.
allowed-tools: Bash(uv *), Bash(pre-commit *)
---

# Pre-commit Checks

Run pre-commit hooks to validate code quality before committing.

## Current Context

**Git Status:**
`!`git status --short``

**Staged Files (first 20):**
`!`git diff --cached --name-only | head -20``

## Execution Options

Choose the appropriate option based on the current context:

1. **All files** (recommended for initial setup or comprehensive checks):
   ```bash
   uv run pre-commit run --all-files
   ```

2. **Staged files only** (faster, for incremental commits):
   ```bash
   uv run pre-commit run
   ```

If `uv` is not available, fall back to:
```bash
pre-commit run --all-files
# or
pre-commit run
```

## On Failure

If any hooks fail:
1. Display the failures clearly
2. Fix the issues automatically when possible
3. Re-run pre-commit to verify fixes

**IMPORTANT:** Pre-commit is MANDATORY before committing - never skip it.
