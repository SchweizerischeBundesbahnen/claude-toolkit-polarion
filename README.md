# SBB Claude Plugins

A monorepo marketplace providing specialized Claude Code plugins for Python, Ansible, containers, and development workflows.

## Available Plugins

| Plugin | Agents | Description |
|--------|--------|-------------|
| **ansible** | 3 | Ansible infrastructure automation, Molecule testing, Testinfra verification |
| **python** | 4 | Python/FastAPI development, API design, debugging, Ruff linting |
| **containers** | 2 | Alpine Linux and Red Hat UBI container optimization |
| **core** | 3 | Documentation lookup, research analysis, workflow commands, hooks |

## Installation

### Add the Marketplace

```bash
/plugin marketplace add ariwk/sbb-polarion-claude-toolkit
```

### Install Individual Plugins

```bash
# Install only what you need
/plugin install ansible@sbb-claude-plugins
/plugin install python@sbb-claude-plugins
/plugin install containers@sbb-claude-plugins
/plugin install core@sbb-claude-plugins
```

### Verify Installation

```bash
/plugins    # Check installed plugins
/agents     # Verify agents are available
```

## Plugin Details

### Ansible Plugin

| Agent | Description |
|-------|-------------|
| **ansible-expert** | Playbooks, roles, collections, debugging |
| **molecule-expert** | Molecule testing framework for Ansible roles |
| **testinfra-expert** | Infrastructure testing with pytest |

### Python Plugin

| Agent | Description |
|-------|-------------|
| **python-developer** | Clean Python, FastAPI, async patterns, PEP standards |
| **api-developer** | REST/GraphQL API design, OpenAPI documentation |
| **code-debugger** | Systematic debugging and root cause analysis |
| **ruff-expert** | Ruff linter configuration |

### Containers Plugin

| Agent | Description |
|-------|-------------|
| **alpine-container-expert** | Alpine Linux optimization and security |
| **ubi-redhat-expert** | Red Hat UBI/OpenShift containers |

### Core Plugin

| Agent | Description |
|-------|-------------|
| **context7** | Documentation assistant using Context7 MCP |
| **deep-research-analyst** | Internet research with critical analysis |
| **subagent-expert** | Claude Code agent creation and review |

**Commands:**
- `/precommit` - Run pre-commit checks
- `/save-todos` - Save TodoWrite state to TODO.md
- `/tox` - Run tox test suite

**Hooks:**
- PostToolUse: Reminds to run pre-commit after file modifications
- PreCompact: Saves todos before context compaction

## Repository Structure

```
sbb-polarion-claude-toolkit/
├── .claude-plugin/
│   └── marketplace.json        # Marketplace manifest
├── plugins/
│   ├── ansible/
│   │   ├── .claude-plugin/plugin.json
│   │   └── agents/
│   ├── python/
│   │   ├── .claude-plugin/plugin.json
│   │   └── agents/
│   ├── containers/
│   │   ├── .claude-plugin/plugin.json
│   │   └── agents/
│   └── core/
│       ├── .claude-plugin/plugin.json
│       ├── agents/
│       ├── commands/
│       └── hooks/
├── CLAUDE.md
└── README.md
```

## Requirements

### Python Development

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
uv tool install ruff mypy pytest
```

### Ansible Development

```bash
uv tool install --with-executables-from ansible-core,ansible-lint,molecule \
    --with 'molecule-plugins[docker]',testinfra ansible
```

### Container Development

```bash
export DOCKER_BUILDKIT=1
```

## License

Apache License 2.0 - See [LICENSE](LICENSE) for details.
