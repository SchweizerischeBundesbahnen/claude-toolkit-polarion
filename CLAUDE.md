# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Claude Code plugin marketplace monorepo** (`claude-toolkit-polarion`) that provides specialized agents, skills, and hooks for Python, Ansible, container development, and core workflows.

## Repository Architecture

### Marketplace Structure

```
claude-toolkit-polarion/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace manifest (owner, plugin list)
├── plugins/
│   ├── ansible/                  # Ansible development toolkit (3 agents)
│   ├── python/                   # Python development toolkit (4 agents)
│   ├── containers/               # Container optimization toolkit (2 agents)
│   └── core/                     # Core utilities (3 agents, skills, hooks)
│       ├── .claude-plugin/
│       │   └── plugin.json       # Plugin metadata
│       ├── agents/*.md           # Agent definitions
│       ├── skills/*/SKILL.md     # Skill definitions
│       ├── hooks/hooks.json      # Hook configurations
│       └── CLAUDE.md             # Redistributed to users installing core plugin
├── .github/workflows/            # CI/CD automation
├── .pre-commit-config.yaml       # Pre-commit hook configuration
├── CLAUDE.md                     # This file (repo development guidance)
└── README.md                     # Public documentation
```

### Key Concepts

**Marketplace vs Plugins:**
- The **marketplace** is a container that lists available plugins in `marketplace.json`
- Each **plugin** is an independent unit with its own `plugin.json`
- Users: `/plugin marketplace add ariwk/claude-toolkit-polarion` then `/plugin install <name>@claude-toolkit-polarion`

**Plugin Components:**
- **Agents** (`.md` files in `agents/`) - AI personas with specialized knowledge and tool access
- **Skills** (`SKILL.md` files in `skills/<name>/`) - User-invocable `/skill` workflows with dynamic context and tool restrictions
- **Hooks** (`hooks/hooks.json`) - Event-driven automation (PostToolUse, PreCompact, etc.)

**CLAUDE.md Files:**
- `CLAUDE.md` (root) - Development guidance for THIS repository
- `plugins/core/CLAUDE.md` - Redistributed to users installing the `core` plugin (contains engineering principles)
- Changes to core plugin's CLAUDE.md should be made there directly

## Plugin Development Workflow

### Creating a New Agent

1. Create `plugins/<plugin-name>/agents/<agent-name>.md`
2. Add YAML frontmatter:
```yaml
---
description: Short description (shown in /agents list)
model: haiku|sonnet|opus  # optional, defaults to parent
tools: [Read, Write, Bash]  # optional, restricts tool access
proactivelyUse: true  # optional, auto-invokes agent for matching tasks
---
```
3. Write agent instructions following the design principles below
4. Test: `/agents` to verify it appears, then invoke manually or via proactivelyUse

**Agent design principles (mandatory):**
- **Minimal and hazard-focused** — cover only what Claude cannot discover itself
- **No code templates** — they anchor Claude to specific patterns even when inappropriate
- **No general best practices** — Claude already knows DRY, TDD, PEP 8, etc.
- **Target under 100 lines** — if you need more, the agent scope is too broad
- **Hazards section** — non-obvious project-specific landmines, not generic advice

### Creating a New Skill

1. Create `plugins/<plugin-name>/skills/<skill-name>/SKILL.md`
2. Add frontmatter:
```yaml
---
name: skill-name
description: Short description (shown in skill list)
allowed-tools: Bash(uv *), Read, Write  # optional, restricts tool access
---
```
3. Write skill instructions in markdown body
4. Use dynamic context with backtick commands: `` `!`git status`` ``
5. Test: `/<skill-name>`

### Modifying Hooks

Edit `plugins/core/hooks/hooks.json`:
- **PostToolUse**: Runs after tool usage (e.g., Edit, Write) - used for pre-commit reminder
- **PreCompact**: Runs before context compaction - used to save TodoWrite state

Example structure:
```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [{"type": "command", "command": "echo 'reminder'"}]
    }],
    "PreCompact": [{
      "matcher": ".*",
      "hooks": [{"type": "command", "command": "echo 'directive'"}]
    }]
  }
}
```

### Adding a New Plugin

1. Create `plugins/<new-plugin>/` directory structure
2. Add `.claude-plugin/plugin.json`:
```json
{
  "name": "plugin-name",
  "description": "Plugin description",
  "author": {
    "name": "SBB Polarion Team",
    "email": "polarion-opensource@sbb.ch"
  },
  "keywords": ["keyword1", "keyword2"]
}
```
3. Update root `marketplace.json` to add the new plugin to the `plugins` array
4. Create agents, skills, or hooks as needed

## Development Commands

### Pre-commit Checks (MANDATORY before commits)

```bash
# Run on all files
uv run pre-commit run --all-files

# Run on staged files only
uv run pre-commit run

# Or use the command (if core plugin installed)
/precommit
```

Pre-commit hooks check:
- Trailing whitespace, EOF fixes
- YAML/JSON validation
- Secret detection (gitleaks)
- GitHub Actions linting (actionlint)
- Conventional commit format (commitizen)

### Git Workflow

```bash
# Standard commit (triggers pre-commit automatically if installed)
git add .
git commit -m "feat: add new feature"

# Push to trigger CI
git push origin <branch>
```

### Verification Commands

If testing with plugins installed locally:
```bash
/plugins    # List installed plugins
/agents     # List available agents
```

## File Formats and Conventions

### Agent Markdown Files

- Location: `plugins/<plugin>/agents/<agent-name>.md`
- Frontmatter: YAML with `description`, optional `model`, `tools`, `proactivelyUse`
- Body: Minimal, hazard-focused instructions (see agent design principles above)

### Skill Markdown Files

- Location: `plugins/<plugin>/skills/<skill-name>/SKILL.md`
- Frontmatter: YAML with `name`, `description`, optional `allowed-tools`
- Body: Step-by-step instructions executed when user runs `/<skill-name>`

### Plugin JSON Schema

```json
{
  "name": "string",
  "description": "string",
  "author": {
    "name": "string",
    "email": "string"
  },
  "keywords": ["array", "of", "strings"]
}
```

### Marketplace JSON Schema

```json
{
  "name": "marketplace-name",
  "owner": {
    "name": "string",
    "email": "string"
  },
  "plugins": [
    {
      "name": "plugin-name",
      "description": "plugin description",
      "source": "./plugins/plugin-name"
    }
  ]
}
```

## CI/CD and Releases

### GitHub Workflows

- **actionlint.yml** - Validates GitHub Actions workflow syntax on all pushes/PRs
- **release-please.yml** - Automated releases on main branch
- **claude.yml** - Claude Code integration (if configured)
- **claude-code-review.yml** - Automated code review (if configured)

### Release Process

This repo uses **release-please** for automated semantic versioning:

1. Make changes using conventional commits:
   - `feat:` - New feature (minor version bump)
   - `fix:` - Bug fix (patch version bump)
   - `BREAKING CHANGE:` - Breaking change (major version bump)
   - `chore:`, `docs:`, `ci:` - No version bump

2. Push to `main` branch
3. release-please bot creates/updates a release PR
4. Merge the release PR to create a new GitHub release and git tag
5. CHANGELOG.md is automatically updated

### Version Pinning

Users can install specific versions:
```bash
/plugin marketplace add ariwk/claude-toolkit-polarion#v1.0.0
/plugin marketplace add ariwk/claude-toolkit-polarion#main
```

## Common Development Tasks

### Testing Changes Locally

1. Make changes to plugin files
2. If you have the marketplace installed: `/plugin reload <plugin-name>`
3. Test agents/commands/hooks

### Adding Context7 Integration to Agents

If an agent needs documentation lookups:

1. Add Context7 MCP tools to agent's tools list:
```yaml
tools: [Read, Write, mcp__plugin_context7_context7__resolve-library-id, mcp__plugin_context7_context7__query-docs]
```

2. In agent instructions, document how to use Context7 for lookups

### Modifying Documentation

- `README.md` - Public-facing user documentation (installation, usage)
- `CLAUDE.md` (root) - Development guidance for this repository
- `plugins/core/CLAUDE.md` - Redistributed to users (engineering principles)
- `CHANGELOG.md` - Auto-generated by release-please, do not edit manually

## Security Notes

This repository is public and open-source:
- Never commit secrets, API keys, or credentials
- Use placeholder emails like `example@example.com` in examples
- Pre-commit hooks check for secrets via gitleaks
- All commits must follow conventional commit format
