---
name: tox
description: Run tox test suite. Use for running tests across environments.
allowed-tools: Bash(uv *), Bash(tox *)
---

# Tox Test Suite

Run tox to execute tests across multiple Python environments.

## Current Context

**Configuration files present:**
`!`ls tox.ini pyproject.toml 2>/dev/null || echo "No tox configuration found"``

## Execution Options

**Basic execution:**
```bash
uv run tox
```

**Common options:**

| Option | Command | Description |
|--------|---------|-------------|
| Specific environment | `uv run tox -e py311` | Run only py311 environment |
| Parallel execution | `uv run tox -p auto` | Run environments in parallel |
| Recreate virtualenvs | `uv run tox -r` | Force recreate test environments |
| List environments | `uv run tox -l` | Show available test environments |
| Verbose output | `uv run tox -v` | More detailed output |

**Combining options:**
```bash
uv run tox -e py311 -v    # Specific env with verbose
uv run tox -p auto -r     # Parallel with recreate
```

## Output Handling

- Display test results clearly
- On failure, show failing tests and error messages
- Summarize pass/fail counts for each environment

## Fallback

If `uv` is not available:
```bash
tox [options]
```
