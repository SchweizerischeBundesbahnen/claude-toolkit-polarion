# Claude Toolkit Polarion - Core Plugin

**NEVER expose SBB internal information** in output, code, docs, or external systems:
- Internal URLs (*.sbb.ch domains, except public ones like www.sbb.ch)
- Internal server names, hostnames, network configurations
- Internal email addresses, employee names, usernames
- Internal project identifiers or ticket numbers
- Use generic examples: `example.com`, `user@example.com`, `PROJECT-123`, `localhost`

**NEVER take shortcuts to make things "work":**
- Never suppress or comment out lint errors — fix the root cause
- Never skip, remove, or weaken a failing test — fix the code until it passes
- Never defer broken things to "later" — finish everything before moving on
- Never add `# type: ignore`, `# noqa`, `@pytest.mark.skip`, or equivalent suppressions
- Every piece of work must be complete, correct, and production-ready
- Before finishing, re-read the original request and verify every part was addressed — do not silently drop requirements

**Verify your work before declaring it done.** Run the project's test suite and linters after code changes. Pre-commit is mandatory — never use `--no-verify`. Run with `/precommit`.

**Use Context7 MCP proactively** for library documentation lookups.

**Code reviews: terse, actionable, changed lines only.** Skip formatting, import order, static analysis, commit messages (automated tools handle these). Focus on security, correctness, breaking changes, resource leaks, missing tests. Format: `file:line` - Problem - Fix.

**NEVER put tool/library versions in documentation** — read from `pyproject.toml`, `package.json`, lock files, or runtime.

**CLAUDE.md** — gotchas and architecture decisions only. Document *why*, not *what*. Follow `/claude-md` skill principles.
**TODO.md** — tactical tasks and progress. Auto-saved before `/compact` (PreCompact hook). Manual: `/save-todos`.
**README.md** — user-facing only. No internal dev details.
**`/docs`** — all additional documentation. Never create standalone `.md` files in the project root.
