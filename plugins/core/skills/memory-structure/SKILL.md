---
name: memory-structure
description: >
  Decides where to put project knowledge, conventions, or workflows in Claude Code projects.
  Routes to the right persistence layer (agent, skill, hook, CLAUDE.md, .claude/rules/) and
  scope (repo, plugin, user). Use when deciding "should this be an agent or skill?", "where
  does this convention go?", "CLAUDE.md or skill?", "which scope?", or when structuring how
  Claude's knowledge is organized across persistence layers.
user-invocable: false
allowed-tools: Read, Write, Glob, Grep, AskUserQuestion
---

# Memory Structure

Help the user decide where to persist knowledge and create it in the right place.

## Current Context

**Existing agents:**
`!`find plugins/*/agents -name '*.md' 2>/dev/null | sort``

**Existing skills:**
`!`find plugins/*/skills -mindepth 1 -maxdepth 1 -type d 2>/dev/null | sort``

**Existing hooks:**
`!`find plugins/*/hooks -name '*.json' 2>/dev/null | sort``

**CLAUDE.md files:**
`!`find . -name 'CLAUDE.md' -not -path './.git/*' 2>/dev/null | sort``

## Decision Framework

Ask what the user wants to achieve, then route:

| The user has... | Mechanism | Why |
|---|---|---|
| A convention that always applies | CLAUDE.md | Always in context, zero invocation cost |
| A convention for specific file types | `.claude/rules/` with `paths` | Loaded only when matching files opened |
| A repeatable workflow with decision points | Skill | On-demand, tool-restricted, dynamic context |
| A need for isolated deep analysis | Agent | Separate context window, proactive matching |
| An automated response to a system event | Hook | Event-driven, no user action needed |

### Scope routing (who sees it?)

| Audience | Where |
|---|---|
| Contributors to this repo only | Root `CLAUDE.md` |
| End-users who install a plugin | `plugins/<name>/CLAUDE.md` |
| Personal preferences | `~/.claude/CLAUDE.md` |

## Gotchas by mechanism

### Agents
- Under 100 lines. If you need more, the scope is too broad.
- No code templates — they anchor Claude to specific patterns even when inappropriate.
- Gotchas and hazards only. Skip anything Claude already knows.
- Claude has ~10K tokens of built-in agent/skill creation guidance. Don't duplicate it.

### Skills
- `allowed-tools` is the point — without tool restrictions, use an agent instead.
- Use `` `!`command`` `` for dynamic context. Skills without it are usually too simple.
- `disable-model-invocation: true` for side-effects (deploy, send-message).
- `user-invocable: false` for background knowledge Claude auto-loads.
- `context: fork` runs as subagent. Don't use if the skill is just guidelines with no actionable task.
- Under 500 lines. Move detail to supporting files.

### CLAUDE.md
- Under 200 lines per file. Longer files reduce adherence and cost more tokens.
- Gotchas and architecture decisions only. Document *why*, not *what*.
- `plugins/core/CLAUDE.md` is redistributed to end-users — changes affect all users.
- When creating or editing CLAUDE.md files, use the `/claude-md` skill — it enforces evidence-based best practices (ETH Zurich AGENTBENCH 2026) to avoid performance degradation.

### Hooks
- Only for automated event responses (PostToolUse, PreCompact, etc.).
- Matcher patterns are regex, not glob.
- Keep hook commands fast — they block the tool call.

## Workflow

1. Ask the user what they want to achieve (use AskUserQuestion if unclear)
2. Route to the right mechanism and scope using the framework above
3. Check for overlap with existing components (see Current Context)
4. Create the file following the conventions for that mechanism
