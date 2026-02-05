# Agent & Skill Testing Framework Research

> Research document for [Issue #6](https://github.com/ariwk/sbb-polarion-claude-toolkit/issues/6) — summarizing existing validation tools and recommending an implementation approach for the `claude-toolkit-polarion` marketplace.

## Problem Statement

This marketplace contains 12 agents and 3 skills across 4 plugins (`ansible`, `python`, `containers`, `core`). Currently there is no automated validation that plugin components:

- Have valid YAML frontmatter with required fields
- Follow Claude Code plugin structure conventions
- Contain well-formed tool configurations
- Meet quality standards (description length, content completeness)
- Avoid security anti-patterns (hardcoded secrets, dangerous commands)

Without validation, regressions can be introduced silently — broken frontmatter, invalid tool names, or missing required fields only surface when a user installs the plugin and encounters errors.

## Existing Tools Discovered

### 1. `claude plugin validate` — Official Anthropic CLI

**Type:** Deterministic structural validation
**Cost:** Free
**CI/CD:** Yes (CLI command)

Validates plugin manifest schema, directory structure, and JSON syntax. Available as part of the Claude Code CLI.

```bash
claude plugin validate ./plugins/your-plugin
```

**What it checks:**
- `plugin.json` schema compliance (required `name` field, valid JSON)
- Directory structure (`.claude-plugin/plugin.json` placement)
- Component path resolution

**Limitations:** Structure only — does not validate agent frontmatter, skill quality, or semantic correctness.

**Source:** [Claude Code Plugins Reference](https://code.claude.com/docs/en/plugins-reference)

---

### 2. `@carlrannaberg/cclint` — Static Plugin Linter (Node.js)

**Type:** Deterministic static analysis
**Cost:** Free (MIT license)
**CI/CD:** Native support with `--fail-on` flag and multiple output formats

The most comprehensive static linter for Claude Code plugins. Validates agents, commands, settings, and documentation.

**Installation:**
```bash
npm install -g @carlrannaberg/cclint
```

**CLI usage:**
```bash
cclint                           # Auto-detect and lint project
cclint agents                    # Agent definitions only
cclint commands                  # Commands only
cclint --format json --output lint-report.json
cclint --fail-on error           # CI mode: exit 1 on errors
cclint --fail-on warning         # Stricter: fail on warnings too
```

**What it validates:**

| Component | Checks |
|-----------|--------|
| Agents (.md) | Required fields (`name`, `description`), tool configurations/syntax, color validation, naming conventions, filename matching, unknown/deprecated field warnings |
| Commands (.md) | Frontmatter schema, tool permissions in `allowed-tools`, Bash command patterns, file reference patterns |
| Settings (.json) | JSON syntax/schema, hook configuration structure, event types, command syntax |
| CLAUDE.md | Required sections, document structure, template compliance |

**Programmatic API:**
```javascript
import CClint from '@carlrannaberg/cclint';
const linter = new CClint();
const summary = await linter.lintProject('./my-project', { quiet: true });
```

**Source:** [carlrannaberg/cclint on GitHub](https://github.com/carlrannaberg/cclint)

---

### 3. `@felixgeelhaar/cclint` — CLAUDE.md Linter (Node.js)

**Type:** Deterministic static analysis
**Cost:** Free (MIT license)
**CI/CD:** GitHub Action available (`felixgeelhaar/cclint@v0.6.0`)

Focused specifically on CLAUDE.md context file validation with 10 built-in rules.

**Installation:**
```bash
npm install -g @felixgeelhaar/cclint
# or without installation:
npx @felixgeelhaar/cclint lint your-claude.md
```

**Key rules:**

| Rule | What It Checks |
|------|----------------|
| Import Syntax | `@path/to/file` syntax, duplicates, max depth (5 hops), circular dependencies |
| Content Organization | Heading hierarchy, bullet points, vague language, specificity |
| Command Safety | Dangerous bash commands, unsafe sudo, error handling |
| File Size | 10,000 character limit (configurable) |
| Structure | Required sections: Project Overview, Development Commands, Architecture |

**GitHub Actions integration:**
```yaml
- name: Lint CLAUDE.md
  uses: felixgeelhaar/cclint@v0.6.0
  with:
    files: 'CLAUDE.md'
```

**Configuration (`.cclintrc.json`):**
```json
{
  "rules": {
    "file-size": {
      "enabled": true,
      "severity": "warning",
      "options": { "maxSize": 15000 }
    }
  },
  "ignore": ["*.backup.md"]
}
```

**Source:** [felixgeelhaar/cclint on GitHub](https://github.com/felixgeelhaar/cclint)

---

### 4. `sjnims/plugin-dev` — Plugin Validator + Skill Reviewer Agents

**Type:** LLM-based (non-deterministic)
**Cost:** Free tool, uses Claude API quota per invocation
**CI/CD:** Not directly — requires interactive Claude Code session

A comprehensive development toolkit with 9 skills, 3 validation agents, and 3 slash commands.

**Agents:**

| Agent | What It Validates |
|-------|-------------------|
| `plugin-validator` | Directory structure, plugin.json completeness, marketplace.json |
| `skill-reviewer` | Trigger phrase reliability, documentation quality, best practices |
| `agent-creator` | Agent file quality during creation |

**Installation:**
```bash
/plugin marketplace add sjnims/plugin-dev
/plugin install plugin-dev@sjnims/plugin-dev
```

**Usage (conversational triggers):**
- "validate my plugin"
- "review my skill"
- "check plugin structure"
- "validate my plugin at plugins/my-plugin"

**Limitations:** Requires an interactive Claude Code session. Results are non-deterministic (LLM-based). Cannot be run as a simple CI command without the Claude Agent SDK.

**Source:** [sjnims/plugin-dev on GitHub](https://github.com/sjnims/plugin-dev)

---

### 5. `sjnims/cc-plugin-eval` — 4-Stage Behavioral Evaluation

**Type:** Hybrid (deterministic analysis + LLM-based evaluation)
**Cost:** ~$1.50–$3.75 per full run (12 agents + 3 skills) with Batch API
**CI/CD:** Yes — supports JUnit XML, TAP, JSON, YAML output formats

The most comprehensive behavioral testing framework. Validates that skills, agents, and commands activate correctly via programmatic detection and LLM judgment.

**4 stages:**

| Stage | Method | What It Does |
|-------|--------|--------------|
| 1. Analysis | Deterministic | Parses plugin structure, extracts trigger phrases |
| 2. Generation | LLM-based | Creates test scenarios (positive/negative/edge cases) |
| 3. Execution | Hybrid | Runs scenarios against Claude Agent SDK |
| 4. Evaluation | Programmatic + LLM | Detects triggering, calculates accuracy metrics |

**Installation:**
```bash
git clone https://github.com/sjnims/cc-plugin-eval.git
npm install && npm run build
echo "ANTHROPIC_API_KEY=sk-ant-..." > .env
```

**CLI usage:**
```bash
# Dry run with cost estimate (no API calls)
npx cc-plugin-eval run -p ./plugin --dry-run

# Full evaluation
npx cc-plugin-eval run -p ./plugin

# Individual stages
npx cc-plugin-eval analyze -p ./plugin
npx cc-plugin-eval generate -p ./plugin
npx cc-plugin-eval execute -p ./plugin
npx cc-plugin-eval evaluate -p ./plugin
```

**Cost controls:**
- `--dry-run` / `--estimate` — shows USD/token projections before execution
- `execution.max_budget_usd` — caps API spending
- `execution.timeout_ms` — prevents runaway executions
- Batch API support — **50% cost savings** via Anthropic Batches API
- Session batching — ~80% reduction in subprocess overhead

**Estimated cost for this project (12 agents + 3 skills):**
- ~5 scenarios per component = 75 scenarios
- At ~$0.02–0.05 per scenario (with batching) = **$1.50–$3.75 per full run**
- With `--fast` mode (reruns only failed): significantly less

**Source:** [sjnims/cc-plugin-eval on GitHub](https://github.com/sjnims/cc-plugin-eval)

---

### 6. Tessl CLI — Skill Lifecycle Platform

**Type:** Hybrid (deterministic lint + LLM-based review)
**Cost:** Free account for basic access; enterprise pricing not publicly documented
**CI/CD:** CLI commands can be scripted; no native GitHub Action

Tessl provides a complete skill lifecycle: creation, validation, publishing, and discovery. Evaluates skills in two ways: **review evals** (structure vs. Anthropic best practices) and **task evals** (real agent performance with/without the skill).

**Installation:**
```bash
npm i -g @tessl/cli
```

**Key commands:**

| Command | What It Does |
|---------|--------------|
| `tessl skill lint ./my-skill` | Validates structure against Agent Skills specification (required files, frontmatter fields) |
| `tessl skill review ./my-skill` | Comprehensive conformance analysis with improvement recommendations |
| `tessl skill publish --workspace myteam ./my-skill` | Publishes with automatic evaluation scoring |
| `tessl skill search <query>` | Search the Tessl registry (2,000+ pre-evaluated skills) |

**What `tessl skill lint` checks:**
- Required files and proper structure
- Frontmatter fields (name, description, etc.)
- Agent Skills specification compliance

**What `tessl skill review` provides:**
- Detailed conformance analysis against Anthropic best practices
- Improvement recommendations
- Best practice alignment scoring

**Sources:**
- [Skills on Tessl Announcement](https://tessl.io/blog/skills-are-software-and-they-need-a-lifecycle-introducing-skills-on-tessl/)
- [Tessl CLI Commands](https://docs.tessl.io/reference/cli-commands)
- [Tessl Documentation](https://docs.tessl.io/)

## Comparison Matrix

| Tool | Validates | Deterministic | CI/CD Ready | Cost | Best For |
|------|-----------|:---:|:---:|------|----------|
| `claude plugin validate` | Structure/manifest | Yes | Yes | Free | Basic structure checks |
| `@carlrannaberg/cclint` | Full plugin (agents, commands, settings) | Yes | Yes | Free | Comprehensive static analysis |
| `@felixgeelhaar/cclint` | CLAUDE.md files | Yes | Yes | Free | Context file quality |
| `sjnims/plugin-dev` | Quality + best practices | No | No | API credits | Manual development-time review |
| `sjnims/cc-plugin-eval` | Triggering behavior | Hybrid | Yes | ~$3/run | Behavioral verification |
| `tessl skill lint/review` | Structure + quality | Hybrid | Partial | Free tier | Publishing workflow |

## Pre-commit Integration

**No existing pre-commit hooks** were found specifically for Claude Code plugin validation. The options are:

1. **GitHub Actions** (recommended) — run cclint as a CI step on every PR
2. **Local pre-commit hook** — wrap cclint in a shell script via `.pre-commit-config.yaml` (custom `local` hook)
3. **npm scripts** — add `"lint:plugins": "cclint --fail-on error"` to a `package.json`

Since this project already uses `pre-commit` for other checks, a custom local hook could be added, but this would require Node.js in the pre-commit environment. GitHub Actions is simpler.

## Recommended Tiered Approach

### Tier 1: Every PR — Static Validation (Free, Deterministic)

**Tools:** `@carlrannaberg/cclint`

Run on every pull request targeting plugin files. Catches structural issues immediately with zero API cost.

```yaml
# .github/workflows/plugin-validate.yml
name: Plugin Validation
on:
  pull_request:
    paths: ["plugins/**"]
  push:
    branches: [main]
    paths: ["plugins/**"]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
      - run: npm install -g @carlrannaberg/cclint
      - name: Lint all plugins
        run: |
          for plugin in plugins/*/; do
            echo "::group::Validating $plugin"
            cclint --root "$plugin" --fail-on error
            echo "::endgroup::"
          done
```

**What this catches:** Invalid frontmatter, missing required fields, bad tool configurations, naming convention violations, deprecated fields.

### Tier 2: Weekly/Pre-release — Behavioral Evaluation (~$3/run)

**Tool:** `sjnims/cc-plugin-eval`

Run weekly on a schedule and on-demand before releases. Validates that agents and skills actually trigger correctly.

```yaml
# .github/workflows/plugin-eval.yml
name: Plugin Behavioral Evaluation
on:
  schedule:
    - cron: '0 0 * * 0'  # Weekly on Sunday
  workflow_dispatch:

jobs:
  evaluate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
      - run: npm install -g cc-plugin-eval
      - name: Run evaluation
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          for plugin in plugins/*/; do
            npx cc-plugin-eval run -p "$plugin" \
              --config .cc-plugin-eval.yaml \
              --format junit \
              --output "reports/$(basename $plugin).xml"
          done
      - uses: actions/upload-artifact@v4
        with:
          name: evaluation-reports
          path: reports/
```

**Configuration (`.cc-plugin-eval.yaml`):**
```yaml
execution:
  max_budget_usd: 5.00
  timeout_ms: 300000
generation:
  scenarios_per_component: 3
  model: claude-sonnet-4-20250514
```

### Tier 3: Optional — Tessl Publishing

If public distribution through the Tessl registry is desired:

```bash
tessl skill publish --workspace sbb-polarion ./plugins/core/skills/precommit
```

This triggers automatic evaluation and makes skills discoverable in the Tessl registry.

## Implementation Roadmap

### Phase 1: Static Validation in CI

1. Create `.github/workflows/plugin-validate.yml` with cclint
2. Run cclint locally and fix any issues found
3. Verify all 12 agents + 3 skills pass validation
4. Merge to main

**Acceptance criteria:**
- [ ] CI workflow runs cclint on all plugins for every PR
- [ ] All 12 agents + 3 skills pass validation
- [ ] Clear error output on failures with GitHub Actions grouping

### Phase 2: Behavioral Evaluation (Future)

1. Add `ANTHROPIC_API_KEY` as a repository secret
2. Create `.github/workflows/plugin-eval.yml` with cc-plugin-eval
3. Create `.cc-plugin-eval.yaml` configuration with budget limits
4. Run initial evaluation and establish baseline metrics
5. Set up weekly schedule

**Acceptance criteria:**
- [ ] Weekly evaluation runs successfully
- [ ] Budget stays within $5/week
- [ ] JUnit XML reports uploaded as artifacts

### Phase 3: Tessl Integration (Optional)

1. Install Tessl CLI
2. Publish skills to registry
3. Evaluate integration into CI pipeline

## Key Architectural Insight: The Deep Trilogy Pattern

The [Deep Trilogy](https://pierce-lamb.medium.com/what-i-learned-while-building-a-trilogy-of-claude-code-plugins-72121823172b) by Pierce Lamb (`/deep-project`, `/deep-plan`, `/deep-implement`) demonstrates the most testable plugin architecture in the community. The core principle:

> **"Respect the boundary between what should be code and what should be Claude."**

- **Deterministic tasks** belong in tested scripts (`scripts/`) — not in SKILL.md
- **State management** belongs in files — not in Claude's memory
- **Recovery logic** belongs in setup sessions (`validate-env.sh`) — not in hope
- **SKILL.md** is a pure orchestrator — it calls scripts and makes decisions based on their output

This makes plugins testable: unit-test the scripts with pytest, validate the environment with shell scripts, and let the SKILL.md handle only what requires LLM judgment. See [`docs/plugin-testing-ecosystem-analysis.md`](./plugin-testing-ecosystem-analysis.md) for the full analysis.

## Limitations and Gaps

1. **No official Anthropic test framework** — `claude plugin validate` is structural only; no semantic or behavioral testing from Anthropic.
2. **LLM-based tools are non-deterministic** — Results may vary between runs. Use for quality assurance, not strict pass/fail gating.
3. **Tessl pricing unclear** — Free tier exists but enterprise/API limits are not publicly documented.
4. **cc-plugin-eval requires API key** — Cannot be used in fork PRs without secrets access.
5. **cclint variants differ** — `@carlrannaberg/cclint` covers full plugins; `@felixgeelhaar/cclint` focuses on CLAUDE.md. Use both if needed.
6. **Most plugins are not designed for testability** — The Deep Trilogy pattern (scripts + orchestrator) is the exception, not the norm. Adopting this pattern for future skills would improve testability significantly.

## Sources

### Official Anthropic
- [Create Plugins - Claude Code Docs](https://code.claude.com/docs/en/plugins)
- [Plugins Reference - Claude Code Docs](https://code.claude.com/docs/en/plugins-reference)
- [claude-plugins-official GitHub](https://github.com/anthropics/claude-plugins-official)

### Community Tools
- [sjnims/plugin-dev](https://github.com/sjnims/plugin-dev) — Plugin validator + skill reviewer agents
- [sjnims/cc-plugin-eval](https://github.com/sjnims/cc-plugin-eval) — 4-stage behavioral evaluation framework
- [carlrannaberg/cclint](https://github.com/carlrannaberg/cclint) — Static plugin linter (full plugin)
- [felixgeelhaar/cclint](https://github.com/felixgeelhaar/cclint) — CLAUDE.md linter

### Tessl
- [Skills on Tessl Announcement](https://tessl.io/blog/skills-are-software-and-they-need-a-lifecycle-introducing-skills-on-tessl/)
- [Tessl CLI Commands](https://docs.tessl.io/reference/cli-commands)
- [Tessl Documentation](https://docs.tessl.io/)

### Deep Trilogy (Plugin Architecture Reference)
- [What I learned building a trilogy of Claude Code Plugins](https://pierce-lamb.medium.com/what-i-learned-while-building-a-trilogy-of-claude-code-plugins-72121823172b) — Core design principles
- [piercelamb/deep-plan](https://github.com/piercelamb/deep-plan) — Planning plugin with pytest, validation scripts, multi-LLM review
- [piercelamb/deep-implement](https://github.com/piercelamb/deep-implement) — TDD implementation with adversarial code review

### Related Research
- [Plugin Testing Ecosystem Analysis](./plugin-testing-ecosystem-analysis.md) — How official and community plugins test
- [Local Model Behavioral Testing](./local-model-behavioral-testing.md) — Free A/B testing with local LLMs

### General Agent Evaluation
- [Demystifying Evals for AI Agents - Anthropic](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)
