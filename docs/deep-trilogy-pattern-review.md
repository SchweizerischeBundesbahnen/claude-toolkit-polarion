# Deep Trilogy Pattern Review: This Repository

> Assessment of whether `claude-toolkit-polarion` follows the Deep Trilogy plugin architecture pattern. February 2026.

## Summary

This marketplace's plugins do **not** follow the Deep Trilogy pattern — and for the current scope, they don't need to. The plugins are **knowledge-only**: they contain expertise and workflow guidance, not deterministic processing. The Deep Trilogy pattern is designed for plugins with multi-phase workflows that perform deterministic work that could fail. Our plugins delegate all execution to Claude's built-in tools.

---

## Deep Trilogy Pattern Compliance

| Deep Trilogy Principle | Our Implementation | Status |
|---|---|:---:|
| Deterministic logic in `scripts/` | No scripts — skills invoke tools directly | Missing |
| SKILL.md = pure orchestrator calling scripts | SKILL.md contains inline bash commands and tool logic | Partial |
| State management via files | Minimal — only `TODO.md` via `/save-todos` | Minimal |
| Pre-execution validation (`validate-env.sh`) | None — no environment checks before skill execution | Missing |
| Resumable workflows with checkpoints | None — no state persistence across sessions | Missing |
| `tests/` directory with pytest | None | Missing |

---

## Current Architecture: Instruction-Driven Agent Pattern

### Structure

```
plugins/
├── core/         # 3 agents, 3 skills, hooks, CLAUDE.md
├── python/       # 4 agents
├── ansible/      # 3 agents
└── containers/   # 2 agents
```

- **12 agents** — Pure instruction documents (120-463 lines of expertise each)
- **3 skills** — Thin tool-invocation wrappers (24-28 lines each)
- **2 hooks** — Simple event-triggered reminders
- **0 scripts** — No shell or Python scripts
- **0 validation logic** — External tools only
- **0 tests** — No test directory

### How It Works

```
User Query → Agent .md (guidance) → Claude invokes tools → Results
```

Agents guide Claude's behavior through detailed instructions. Skills wrap common tool invocations into `/slash` commands. Hooks remind the user about pre-commit checks. All actual execution happens through Claude's built-in tools (Bash, Read, Write, etc.).

---

## Component Audit

### Skills

All three skills contain **inline deterministic logic with no external scripts**:

| Skill | Lines | What It Does | Script Calls |
|-------|------:|--------------|:---:|
| `/precommit` | 24 | Runs `uv run pre-commit` via Bash | None |
| `/save-todos` | 28 | Persists TodoWrite state to `TODO.md` via Read/Write | None |
| `/tox` | 27 | Runs `uv run tox` via Bash | None |

### Hooks

| Hook | Trigger | Action |
|------|---------|--------|
| PostToolUse (Edit\|Write) | After file modifications | Echo reminder to run `/precommit` |
| PreCompact | Before context compaction | Directive to run `/save-todos` |

### Agents

All 12 agents are **pure expertise documents** containing:
- Workflow instructions and best practices
- Tool usage guidance (which tools to use when)
- Code examples as reference (not for execution)
- Coordination patterns (e.g., ansible-expert coordinates with molecule-expert)

No agent contains inline code execution logic, state management, or configuration files.

### State Management

- `TODO.md` — Saved manually via `/save-todos` or automatically before `/compact`
- Format: Markdown with checkboxes (`[ ]`, `[~]`, `[x]`)
- No other persistence mechanisms

---

## Why This Is Appropriate (For Now)

Pierce Lamb's principle — *"Respect the boundary between what should be code and what should be Claude"* — applies when a plugin does deterministic work that could fail:

- File parsing → put in a tested script
- API connectivity checks → put in `validate-env.sh`
- State transitions → persist to files
- Multi-step workflows → add checkpoints for resumability

Our plugins don't do any of that. They teach Claude *how* to use tools it already has. The instruction-driven pattern is the right choice for knowledge-only plugins.

---

## Where the Deep Trilogy Pattern Would Help

If skills grow beyond simple tool invocation, adopting the pattern would improve reliability and testability:

### Pre-Execution Validation

| Skill | Potential Script | What It Would Check |
|-------|-----------------|---------------------|
| `/precommit` | `scripts/checks/validate-precommit.sh` | Verify `pre-commit` is installed, `.pre-commit-config.yaml` exists, `uv` is available |
| `/tox` | `scripts/checks/validate-tox.sh` | Verify `tox.ini` or `[tool.tox]` in `pyproject.toml` exists, `uv` is available |
| `/save-todos` | `scripts/checks/validate-todos.sh` | Verify TodoWrite has tasks before attempting save |

### Example: Converting `/precommit` to Deep Trilogy Pattern

**Current (inline):**
```markdown
---
name: precommit
description: Run pre-commit on all files or modified files
allowed-tools: Bash(uv run pre-commit *)
---
Run pre-commit checks. Show git status first...
```

**Deep Trilogy pattern (orchestrator + script):**
```markdown
---
name: precommit
description: Run pre-commit on all files or modified files
allowed-tools: Bash(${CLAUDE_PLUGIN_ROOT}/scripts/*)
---
1. Run `${CLAUDE_PLUGIN_ROOT}/scripts/checks/validate-precommit.sh`
2. If validation fails, report the issue and stop
3. Ask user: all files or staged only?
4. Run `${CLAUDE_PLUGIN_ROOT}/scripts/run-precommit.sh [--all-files|--staged]`
5. Report results
```

With `scripts/checks/validate-precommit.sh`:
```bash
#!/bin/bash
# Deterministic validation — testable with pytest/bats
command -v uv >/dev/null 2>&1 || { echo "ERROR: uv not found"; exit 1; }
[ -f .pre-commit-config.yaml ] || { echo "ERROR: .pre-commit-config.yaml not found"; exit 1; }
echo "OK: pre-commit environment validated"
```

This script can be unit-tested independently. The SKILL.md only orchestrates.

---

## Recommendations

### Keep Current Pattern For

- All 12 agents — they are pure expertise documents and don't benefit from scripts
- Simple skills that wrap a single command (`/tox`, `/precommit` in current form)
- Hooks — they are already minimal reminders

### Adopt Deep Trilogy Pattern When

- A skill grows beyond ~30 lines of instructions
- A skill performs multi-step deterministic workflows
- A skill needs to validate environment state before execution
- A skill manages state across sessions (e.g., tracking progress through a plan)
- Reliability matters — a script that validates before running prevents confusing mid-workflow failures

### Immediate Low-Effort Wins

| Action | Effort | Impact |
|--------|--------|--------|
| Add `scripts/checks/validate-precommit.sh` | Low | Prevents confusing errors when pre-commit isn't installed |
| Add `scripts/checks/validate-tox.sh` | Low | Prevents confusing errors when tox config is missing |
| Add `tests/` directory with pytest for future scripts | Low | Establishes the pattern for when scripts are added |

---

## Sources

- [What I learned building a trilogy of Claude Code Plugins](https://pierce-lamb.medium.com/what-i-learned-while-building-a-trilogy-of-claude-code-plugins-72121823172b) — Core design principles
- [piercelamb/deep-plan](https://github.com/piercelamb/deep-plan) — Reference implementation of the pattern
- [piercelamb/deep-implement](https://github.com/piercelamb/deep-implement) — TDD + adversarial review implementation
- [Plugin Testing Ecosystem Analysis](./plugin-testing-ecosystem-analysis.md) — Full Deep Trilogy analysis
- [Skill Testing Research](./skill-testing-research.md) — Validation tools and CI recommendations
