# Plugin Testing Ecosystem Analysis

> How official Anthropic and community Claude Code plugins test their implementations. Research conducted February 2026.

## Executive Summary

**Official Anthropic Claude Code plugins have no automated testing.** Neither `anthropics/claude-plugins-official` (28 plugins) nor `anthropics/claude-code` (5 internal plugins) contain test suites, testing frameworks, or CI/CD pipelines that validate plugin functionality. Quality assurance relies on internal code review, the `claude plugin validate` CLI command, and developer-facing validation shell scripts shipped in the `plugin-dev` plugin.

Community plugins show more diversity: `piercelamb/deep-*` (the "Deep Trilogy") is the most mature example — using pytest, validation scripts, and TDD-enforced implementation with a clear separation between deterministic tested code and LLM orchestration. `obra/superpowers` implements behavioral tests using `claude -p` prompt mode with a custom assertion library, and `jeremylongshore/claude-code-plugins-plus-skills` uses Vitest + Docker for e2e testing.

**Our repository's existing pre-commit setup already exceeds what official Anthropic plugins do.** The tiered approach proposed in [`docs/skill-testing-research.md`](./skill-testing-research.md) is more rigorous than anything found in the official ecosystem.

---

## Official Anthropic Plugins

### `anthropics/claude-plugins-official`

**Repository:** [github.com/anthropics/claude-plugins-official](https://github.com/anthropics/claude-plugins-official)

| Aspect | Finding |
|--------|---------|
| Test directories | None across all 28 plugins |
| CI/CD workflows | Single workflow: `close-external-prs.yml` (auto-closes PRs from non-Anthropic contributors) |
| Testing frameworks | None |
| Pre-commit hooks | None |
| Validation scripts | None at repository level |

**Plugins examined:**

| Plugin | Contents | Tests |
|--------|----------|:-----:|
| `typescript-lsp` | README.md only | None |
| `pyright-lsp` | README.md only | None |
| `security-guidance` | `.claude-plugin/`, `hooks/` | None |
| `code-review` | `.claude-plugin/`, `commands/`, README.md | None |
| `example-plugin` | `.claude-plugin/plugin.json`, `.mcp.json`, `commands/`, `skills/`, README.md | None |
| `hookify` | `agents/`, `commands/`, `core/`, `examples/`, `hooks/`, `matchers/`, `skills/`, `utils/` | None |

### `anthropics/claude-code` (Internal Plugins)

**Repository:** [github.com/anthropics/claude-code](https://github.com/anthropics/claude-code)

| Aspect | Finding |
|--------|---------|
| Test directories | None within `plugins/` |
| CI/CD workflows | 11 workflows, all focused on issue management (triage, deduplication, stale closing). Zero plugin testing workflows |
| Testing frameworks | None for plugins |

**Plugins examined:**

| Plugin | Contents | Tests |
|--------|----------|:-----:|
| `plugin-dev` | 3 agents, 7 skills, 6 validation shell scripts | Validation scripts only (see below) |
| `code-review` | `.claude-plugin/`, `commands/code-review.md` | None |
| `feature-dev` | `agents/` (3 agents), `commands/`, README.md | None |
| `security-guidance` | `.claude-plugin/`, `hooks/` | None |

### The `plugin-dev` Plugin — Closest to a Testing Framework

Located at `anthropics/claude-code/plugins/plugin-dev/`, this is the most significant testing-related artifact in the official ecosystem. It provides both shell-based validation scripts and LLM-based validation agents.

**Shell-based validation scripts (deterministic):**

| Script | What It Validates |
|--------|-------------------|
| `validate-agent.sh` | YAML frontmatter presence/closure, required fields (`name`, `description`, `model`, `color`), name format (alphanumeric + hyphens, 3-50 chars), description quality (`<example>` blocks, "Use this agent when" pattern), model validity, system prompt length (20-10000 chars), second-person voice |
| `validate-hook-schema.sh` | JSON syntax, valid event types, required fields (`matcher`, `hooks` array), hook type validity (`command`/`prompt`), type-specific fields, hardcoded path detection, timeout ranges (5-600s) |
| `validate-settings.sh` | Plugin settings file validation |
| `parse-frontmatter.sh` | YAML frontmatter extraction utility |
| `test-hook.sh` | Hook testing with sample input |
| `hook-linter.sh` | Hook script best practices |

**LLM-based validation agents (non-deterministic):**

| Agent | What It Validates |
|-------|-------------------|
| `plugin-validator` | 10-step comprehensive validation: manifest, directory structure, commands, agents, skills, hooks, MCP configuration, file organization, security checks. Outputs structured report with severity (critical/major/minor). Uses `Read`, `Grep`, `Glob`, `Bash` tools. |
| `skill-reviewer` | Skill documentation quality, trigger phrase reliability, best practices alignment |

**Key limitation:** These are developer tools designed for use within an active Claude Code session. They are **not** automated CI tests — they require manual invocation.

### `claude plugin validate` CLI Command

The only official deterministic CI-ready validation tool from Anthropic.

```bash
claude plugin validate ./plugins/your-plugin
```

**What it checks:**
- `plugin.json` JSON syntax and schema compliance
- Required `name` field presence
- Directory structure placement (`.claude-plugin/plugin.json`)
- Component path resolution

**What it does not check:**
- Agent/skill frontmatter validity
- Tool configuration correctness
- Content quality
- Security patterns

---

## Community Plugins

### `piercelamb/deep-*` (Deep Trilogy) — Tested Scripts + TDD + pytest

**Repositories:**
- [piercelamb/deep-project](https://github.com/piercelamb/deep-project) — Transforms vague ideas into plannable components
- [piercelamb/deep-plan](https://github.com/piercelamb/deep-plan) — Transforms components into implementation plans via research, interviews, multi-LLM review
- [piercelamb/deep-implement](https://github.com/piercelamb/deep-implement) — Implements plans with TDD, adversarial code review, atomic commits

**Articles:**
- [What I learned while building a trilogy of Claude Code Plugins](https://pierce-lamb.medium.com/what-i-learned-while-building-a-trilogy-of-claude-code-plugins-72121823172b)
- [The Deep Trilogy: Claude Code Plugins for Writing Good Software, Fast](https://pierce-lamb.medium.com/the-deep-trilogy-claude-code-plugins-for-writing-good-software-fast-33b76f2a022d)

| Aspect | Finding |
|--------|---------|
| Test directories | `tests/` in each plugin — pytest suites covering workflow phases and state management |
| Testing framework | **pytest** (Python 3.11+, managed via `uv`) |
| CI/CD workflows | Not found in repos — tests are developer-run |
| Validation scripts | `scripts/checks/` (setup validation), `scripts/lib/` (shared utils), `scripts/tools/` (state management) |
| What's tested | State persistence/resumption, spec file parsing, output artifact generation, LLM response parsing, environment validation |

**The most important plugin architecture insight in the ecosystem.** Pierce Lamb articulates a core principle that no other plugin author has stated as clearly:

> **"Respect the boundary between what should be code and what should be Claude."**

The three rules:
1. **Deterministic tasks belong in tested scripts** — not in SKILL.md markdown
2. **State management belongs in files** — not in Claude's memory
3. **Recovery logic belongs in setup sessions** — not in hope

**Why this matters for testability:** SKILL.md acts as an **orchestrator** that calls tested scripts, not a logic file. Packing deterministic logic into markdown instructions exposes the system to Claude hallucinating or skipping steps. Instead:

- Deterministic operations live in `scripts/` as Python/Bash
- Scripts report results back to Claude via **print statements**
- Claude acts as **judge/orchestrator** — deciding what to do with the results
- The scripts are unit-tested with pytest; the orchestration is validated by the workflow

**Plugin structure (shared across all three):**
```
.claude-plugin/plugin.json     # Plugin metadata
skills/<name>/SKILL.md         # Orchestrator — calls scripts, not logic
scripts/
  checks/                      # Pre-execution validation (validate-env.sh)
  lib/                         # Shared utilities
  tools/                       # State management scripts
  hooks/                       # Hook implementations
hooks/hooks.json               # Session lifecycle hooks
tests/                         # pytest suite
config.json                    # Plugin configuration
pyproject.toml                 # Python deps managed by uv
```

**Pre-execution validation (`validate-env.sh`):**
- Verifies environment variables (API keys, credentials)
- Tests LLM connectivity
- Checks git state (clean working tree, correct branch)
- Validates directory structure
- Detects pre-commit hooks and formatters
- Runs **before any LLM call** to prevent mid-workflow failures

**TDD enforcement (deep-implement):**
1. For each section: write tests first, then implementation (max 3 retry cycles)
2. Claude reviews its own diff with instructions to be **adversarial** — find edge cases, security issues, logic errors
3. Findings triaged as: ask user / auto-fix / let go
4. Fixes applied, tests run again, only **then** it commits
5. One atomic commit per section — prevents mixing concerns

**Resumable state management:**
- File-based persistence (`deep_plan_config.json`, section manifests, saved commit hashes)
- Re-running detects completed steps and resumes automatically
- Idempotent execution — running twice with same input safely continues work
- Context monitoring prompts users to compact every 2 sections

**Tradeoffs:**
- Tests cover deterministic scripts, not LLM orchestration behavior
- Requires Python 3.11+ and uv
- No CI/CD pipeline — tests are run locally during development
- Most sophisticated testing approach found in the community

---

### `obra/superpowers` — Behavioral Tests with `claude -p`

**Repository:** [github.com/obra/superpowers](https://github.com/obra/superpowers)

| Aspect | Finding |
|--------|---------|
| Test directories | `tests/` with 5 subdirectories: `claude-code/`, `explicit-skill-requests/`, `opencode/`, `skill-triggering/`, `subagent-driven-dev/` |
| Testing framework | Custom shell-based test harness |
| Key files | `run-skill-tests.sh`, `test-helpers.sh` |
| What's tested | Skill triggering behavior, workflow ordering, self-review requirements, plan reading efficiency, reviewer skepticism, branch safety |

**Testing approach — the most sophisticated in the community:**

The `test-helpers.sh` provides a custom assertion library:

| Function | Purpose |
|----------|---------|
| `run_claude` | Runs Claude Code in prompt mode (`claude -p`) with timeout and tool restrictions |
| `assert_contains` | Verifies output includes a pattern |
| `assert_not_contains` | Validates pattern absence |
| `assert_count` | Confirms pattern appears exactly N times |
| `assert_order` | Checks pattern A appears before pattern B |
| `create_test_project` | Creates temporary test directory scaffolding |
| `cleanup_test_project` | Tears down test directory |
| `create_test_plan` | Scaffolds test implementation plans |

**Test execution model:**
- **Fast tests** (~2 min): Validate skill instructions load and Claude follows expected workflows
- **Integration tests** (~10-30 min): Execute complete workflows with real project scaffolding
- Tests run via `claude -p` (prompt mode), making Claude the test execution engine
- Results are assertion-based with `[PASS]`/`[FAIL]` markers
- Supports `--verbose`, `--timeout`, `--integration` flags
- Exit codes support CI/CD integration

**Tradeoffs:**
- Requires API credits for every test run
- Non-deterministic — LLM behavior varies between runs
- Slow (minutes to tens of minutes)
- Tests actual behavior, not just structure

### `jeremylongshore/claude-code-plugins-plus-skills` — Vitest + Docker

**Repository:** [github.com/jeremylongshore/claude-code-plugins-plus-skills](https://github.com/jeremylongshore/claude-code-plugins-plus-skills)

| Aspect | Finding |
|--------|---------|
| Test directories | `tests/e2e/` |
| Testing framework | **Vitest** (TypeScript) with `vitest.config.ts` |
| CI/CD | GitHub Actions workflows |
| Infrastructure | Docker-based test environments (`Dockerfile.test`, `docker-compose.test.yml`) |
| What's tested | Installation lifecycle, skill activation, MCP communication |

**Testing approach:**
- Uses test pyramid with isolated temporary directories (`/tmp/claude-e2e-test-{uuid}`)
- Arrange-Act-Assert pattern
- Error path coverage
- The most "traditional" testing setup found in the ecosystem

**Tradeoffs:**
- Tests plugin system mechanics, not LLM behavior
- Requires Docker for full test suite
- Deterministic and free (no API credits)

### `brennercruvinel/CCPlugins` — No Formal Testing

**Repository:** [github.com/brennercruvinel/CCPlugins](https://github.com/brennercruvinel/CCPlugins)

No formal test suite. Quality mechanisms are built into the commands themselves — e.g., `/refactor validate` has built-in validation phases, and git checkpoints are created before destructive operations.

### `kivilaid/plugin-marketplace` — Agent-Based Review

**Repository:** [github.com/kivilaid/plugin-marketplace](https://github.com/kivilaid/plugin-marketplace)

No formal test suite. Quality is maintained through post-write validation via hooks, specialized review agents, and security scanning built into workflows.

---

## Comparison Matrix

| Repository | Type | Tests | Framework | CI Validation | Static Checks | Behavioral Checks |
|------------|------|:-----:|-----------|:-------------:|:-------------:|:-----------------:|
| `anthropics/claude-plugins-official` | Official | No | None | None | None | None |
| `anthropics/claude-code/plugins` | Official | No | None | None | Shell scripts (plugin-dev) | plugin-validator agent |
| `claude plugin validate` CLI | Official | N/A | Built-in | Yes | JSON/schema only | No |
| **`piercelamb/deep-*`** | **Community** | **Yes** | **pytest + bash scripts** | **No** | **validate-env.sh, state checks** | **TDD + adversarial review** |
| `obra/superpowers` | Community | Yes | Custom shell + `claude -p` | Partial | Assertion-based output | Claude as test oracle |
| `jeremylongshore/...` | Community | Yes | Vitest + Docker | Yes | E2E with isolation | No |
| `brennercruvinel/CCPlugins` | Community | No | None | None | In-command validation | No |
| `kivilaid/plugin-marketplace` | Community | No | None | None | Hook-based validation | Agent-based review |

---

## Why Official Plugins Have No Testing

1. **Plugin nature:** Most official plugins are purely markdown and JSON with no executable code. However, the Deep Trilogy demonstrates that well-designed plugins **can** be testable by extracting deterministic logic into `scripts/` and testing those with pytest — leaving SKILL.md as a pure orchestrator. The official plugins simply haven't adopted this pattern.

2. **Non-deterministic behavior:** Agents and skills influence Claude's behavior through prompts. Testing whether a prompt "works correctly" requires running the LLM, which is inherently non-deterministic, expensive, and slow.

3. **Structural simplicity:** Most official plugins are structurally trivial. The `pyright-lsp` plugin is literally just a README.md. The validation of "does this JSON parse correctly" is handled by Claude Code's plugin loader at runtime.

4. **Quality-through-review:** Only Anthropic team members can contribute to official plugins (external PRs are auto-closed). Quality is maintained through internal code review, not automated testing.

5. **Validation is deferred to tooling:** Instead of testing each plugin, Anthropic built the `plugin-dev` plugin with validation utilities and a `plugin-validator` agent. The philosophy is: give developers tools to validate their own plugins.

---

## Ecosystem Patterns Summary

The ecosystem has converged on a three-tier validation model:

### Tier 1 — Structural/Static (deterministic, free, fast)

- `claude plugin validate` — Official CLI command
- `@carlrannaberg/cclint` — Community static linter
- `validate-agent.sh`, `validate-hook-schema.sh` — Official shell scripts in `plugin-dev`
- Pre-commit hooks (YAML/JSON validation, trailing whitespace)

### Tier 2 — Behavioral (non-deterministic, costs money, slow)

- `obra/superpowers` test harness — Custom shell + `claude -p` with assertions
- `sjnims/cc-plugin-eval` — 4-stage evaluation framework (~$1.50-3.75/run)
- `tessl skill lint/review` — Hybrid structural + LLM review

### Tier 3 — E2E/Integration (deterministic infrastructure testing)

- `jeremylongshore/...` — Vitest + Docker for installation lifecycle, MCP communication
- Tests the plugin system mechanics, not the LLM behavior

---

## Recommendations for This Repository

### What We Already Do Better Than Official Plugins

Our existing pre-commit setup includes:
- Trailing whitespace and EOF fixes
- YAML/JSON validation
- Secret detection (gitleaks)
- GitHub Actions linting (actionlint)
- Conventional commit format (commitizen)

This already exceeds what **any** official Anthropic plugin repository does.

### Recommended Next Steps

1. **Add `@carlrannaberg/cclint` to CI** (Tier 1) — This catches agent frontmatter issues, tool configuration errors, and naming convention violations that our current pre-commit hooks do not cover. See the workflow example in [`docs/skill-testing-research.md`](./skill-testing-research.md#tier-1-every-pr--static-validation-free-deterministic).

2. **Adopt the Deep Trilogy pattern for complex skills** — When a skill involves deterministic logic (file parsing, state management, validation), extract it into `scripts/` and test with pytest. Keep SKILL.md as a pure orchestrator. This is the most impactful architectural insight from the ecosystem. See [Pierce Lamb's lessons learned](https://pierce-lamb.medium.com/what-i-learned-while-building-a-trilogy-of-claude-code-plugins-72121823172b).

3. **Add pre-execution validation scripts** — Following the Deep Trilogy's `validate-env.sh` pattern, add setup scripts that verify environment and plugin state before any LLM calls. These are cheap to write, easy to test, and prevent mid-workflow failures.

4. **Consider the `obra/superpowers` pattern for behavioral tests** — If behavioral correctness matters (e.g., ensuring the `precommit` skill triggers on the right phrase), adapt their `claude -p` + assertion approach for a small set of smoke tests.

5. **Use local models for development-time behavioral testing** — Run A/B tests (with/without agent context) on local LLMs to verify agent instructions produce the intended behavioral changes. See [`docs/local-model-behavioral-testing.md`](./local-model-behavioral-testing.md).

### Priority Order

| Priority | Action | Cost | Impact |
|----------|--------|------|--------|
| 1 | Add cclint to GitHub Actions | Free | Catches 80% of structural issues |
| 2 | Extract deterministic logic to `scripts/` + pytest (Deep Trilogy pattern) | Free | Makes plugins testable by design |
| 3 | Add `validate-env.sh` pre-execution checks | Free | Prevents mid-workflow failures |
| 4 | Local model A/B behavioral tests | Free (local hardware) | Validates agent instructions work |
| 5 | `obra/superpowers`-style smoke tests for critical skills | ~$0.10/run | Validates behavioral correctness via Claude |
| 6 | Set up `cc-plugin-eval` for weekly runs | ~$3/run | Full behavioral regression testing |

---

## Sources

### Official Anthropic
- [anthropics/claude-plugins-official](https://github.com/anthropics/claude-plugins-official) — Official plugin directory (28 plugins)
- [anthropics/claude-code](https://github.com/anthropics/claude-code) — Claude Code repository (5 internal plugins)
- [claude-code plugins README](https://github.com/anthropics/claude-code/blob/main/plugins/README.md) — Official plugin documentation
- [Plugins Reference - Claude Code Docs](https://code.claude.com/docs/en/plugins-reference) — CLI commands and debugging

### Community Plugins
- [piercelamb/deep-project](https://github.com/piercelamb/deep-project) — Idea decomposition with resumable state
- [piercelamb/deep-plan](https://github.com/piercelamb/deep-plan) — Planning with research, interviews, multi-LLM review
- [piercelamb/deep-implement](https://github.com/piercelamb/deep-implement) — TDD implementation with adversarial code review
- [What I learned building a trilogy of Claude Code Plugins](https://pierce-lamb.medium.com/what-i-learned-while-building-a-trilogy-of-claude-code-plugins-72121823172b) — Core design principles
- [The Deep Trilogy](https://pierce-lamb.medium.com/the-deep-trilogy-claude-code-plugins-for-writing-good-software-fast-33b76f2a022d) — Architecture overview
- [obra/superpowers](https://github.com/obra/superpowers) — Behavioral tests using `claude -p`
- [jeremylongshore/claude-code-plugins-plus-skills](https://github.com/jeremylongshore/claude-code-plugins-plus-skills) — Vitest + Docker e2e
- [brennercruvinel/CCPlugins](https://github.com/brennercruvinel/CCPlugins) — In-command validation
- [kivilaid/plugin-marketplace](https://github.com/kivilaid/plugin-marketplace) — Agent-based review

### Validation Tools
- [carlrannaberg/cclint](https://github.com/carlrannaberg/cclint) — Static plugin linter
- [sjnims/cc-plugin-eval](https://github.com/sjnims/cc-plugin-eval) — Behavioral evaluation framework
- [sjnims/plugin-dev](https://github.com/sjnims/plugin-dev) — Plugin validator + skill reviewer agents

### Related Research
- [Skill Testing Research](./skill-testing-research.md) — Validation tools overview and CI workflow examples
- [Local Model Behavioral Testing](./local-model-behavioral-testing.md) — Free A/B testing with local LLMs on Apple Silicon
