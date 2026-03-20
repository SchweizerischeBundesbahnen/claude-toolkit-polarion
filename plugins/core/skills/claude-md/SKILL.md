---
name: claude-md
description: This skill should be used when the user asks to "create a CLAUDE.md", "update CLAUDE.md", "add to CLAUDE.md", "edit CLAUDE.md", "write a CLAUDE.md", "improve CLAUDE.md", or mentions setting up agent instructions, context files for Claude, or AGENTS.md. Enforces evidence-based best practices to avoid performance degradation and unnecessary token cost.
version: 1.0.0
---

# CLAUDE.md Best Practices

Apply these rules every time a CLAUDE.md file is created or modified. They are grounded in empirical research (ETH Zurich AGENTBENCH 2026, Lulla et al. 2026) showing that poorly written context files reduce agent performance in the majority of cases and increase inference costs by 20–23%.

See `references/research-findings.md` for the full evidence base.

## The Prime Directive

**Before writing any line, ask: "Can the agent discover this on its own by reading the code?"**

If yes → do not write it. Delete it if it already exists.

Agents navigate codebases effectively without help. What they cannot discover are non-obvious facts that exist outside the code itself.

## What Belongs in CLAUDE.md

Include only **gotchas** — facts the agent would trip over without being told:

1. **Deprecated patterns still in the codebase** — e.g., "The `auth/` module uses the old session API; do not extend it, use `auth-v2/` instead."
2. **Custom or unusual tooling preferences** — e.g., "Use `uv` instead of `pip`; never use `pip install` directly."
3. **Unexpected dependency locations** — e.g., "Production config lives in `infra/prod/` not in the app."
4. **Non-obvious architectural decisions** — e.g., "All DB writes go through the event bus, never directly."
5. **Gotchas that caused agent failures in the past** — add an entry after an agent fails unexpectedly; remove it once the underlying code issue is fixed.

## What Does NOT Belong in CLAUDE.md

Remove or do not add:

- **Codebase overviews or summaries** — 100% of LLM-generated CLAUDE.md files in research contained redundant overviews that hurt performance.
- **Directory structure listings** — agents read the filesystem directly.
- **Technology stack descriptions** — visible from `package.json`, `pyproject.toml`, imports, etc.
- **General coding style** — belongs in linters/formatters, not prose.
- **How to run tests** — agents read `Makefile`, `package.json` scripts, CI config.
- **Navigation hints** — "controllers are in `src/controllers/`" — agents find this themselves.
- **Anything already in a config file, README, or discoverable from code**.

## Length and Structure Rules

- **Keep it short.** Agents exhibit U-shaped attention: they read the start and end, but ignore the middle of long files. A 10-line CLAUDE.md outperforms a 200-line one.
- **No sections for the sake of structure.** If you only have 3 bullets, use 3 bullets — not 3 headers with 1 bullet each.
- **Put the most critical facts first and last.**
- **One fact per line.** Dense prose is harder for agents to anchor on than crisp bullet points.

## Maintenance Model

Treat CLAUDE.md like a bug tracker, not a wiki:

- **Add an entry** when an agent fails unexpectedly due to a non-obvious constraint.
- **Remove an entry** when the underlying issue is resolved in the code itself.
- **Never bulk-generate** CLAUDE.md content from an AI — it will produce codebase overviews that hurt performance. Write entries manually, one gotcha at a time.

## Editing Workflow

When asked to create or update a CLAUDE.md:

1. **Read the existing file** if one exists.
2. **Audit every existing line** against the Prime Directive. Propose removing anything the agent can discover from the code.
3. **For new content**, confirm it is a genuine gotcha (non-discoverable, agent-trip-prone fact).
4. **Keep the file short** — if it grows beyond ~20 lines, scrutinize every entry.
5. **Never add** project overviews, directory tours, or tech stack lists.

## Additional Resources

- **`references/research-findings.md`** — Full research summary with citations (ETH Zurich, Lulla et al., Addy Osmani)
- **Source article**: https://sulat.com/p/agents-md-hurting-you
