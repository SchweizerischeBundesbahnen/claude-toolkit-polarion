# SBB Polarion Claude Toolkit

A comprehensive Claude Code plugin providing specialized agents, workflow commands, and engineering best practices for Python, Ansible, API, and container development.

## Features

### 12 Specialized Agents

| Agent | Description |
|-------|-------------|
| **subagent-expert** | Expert advisor on Claude Code agents - creation, review, validation |
| **context7** | Context-aware documentation assistant using Context7 MCP |
| **deep-research-analyst** | Comprehensive internet research with critical analysis |
| **code-debugger** | Systematic debugging and root cause analysis |
| **python-developer** | Clean Python code following PEP standards, FastAPI specialist |
| **api-developer** | REST/GraphQL API design with OpenAPI documentation |
| **ruff-expert** | Ruff Python linter configuration expert |
| **ansible-expert** | Ansible playbooks, roles, and collections development |
| **molecule-expert** | Molecule testing framework for Ansible roles |
| **testinfra-expert** | TestInfra infrastructure testing with pytest |
| **alpine-container-expert** | Alpine Linux container optimization and security |
| **ubi-redhat-expert** | Red Hat UBI container optimization |

### 3 Workflow Commands

| Command | Description |
|---------|-------------|
| `/precommit` | Run pre-commit checks on all or modified files |
| `/save-todos` | Save TodoWrite state to TODO.md for persistence |
| `/tox` | Run tox test suite using uv |

### Hooks

- **PostToolUse (Edit\|Write)**: Reminds to run `/precommit` after file modifications
- **PreCompact**: Saves TodoWrite state to TODO.md before context compaction

### Engineering Principles (CLAUDE.md)

Always-active guidance loaded into every conversation:

- DRY (Don't Repeat Yourself)
- Single Responsibility Principle
- Configuration-First Design
- Performance Measurement (Never Speculate)
- Test-Driven Development (TDD)
- Spec-Driven Development (SDD)

## Installation

### Prerequisites

- Claude Code CLI installed
- GitHub authentication configured (for private repo access)

### Install from Private Repository

```bash
# Add the private marketplace (requires GitHub auth)
/plugin marketplace add ariwk/sbb-polarion-claude-toolkit

# Install the plugin
/plugin install sbb-polarion-claude-toolkit@ariwk

# Enable it
/plugin enable sbb-polarion-claude-toolkit@ariwk
```

### Verify Installation

```bash
# Check installed plugins
/plugins

# Verify agents are available
/agents
```

## Usage

### Using Agents

Agents are automatically invoked based on task context. You can also explicitly request them:

```
Use the python-developer agent to create a FastAPI endpoint for user authentication.
```

```
Use the alpine-container-expert to optimize my Dockerfile.
```

### Using Commands

```
/precommit          # Run pre-commit on all files
/save-todos         # Save current tasks to TODO.md
/tox                # Run tox tests
```

## Plugin Structure

```
sbb-polarion-claude-toolkit/
├── .claude-plugin/
│   └── plugin.json              # Plugin manifest
├── agents/                      # 12 specialized agents
│   ├── subagent-expert.md
│   ├── context7.md
│   ├── deep-research-analyst.md
│   ├── code-debugger.md
│   ├── python-developer.md
│   ├── api-developer.md
│   ├── ruff-expert.md
│   ├── ansible-expert.md
│   ├── molecule-expert.md
│   ├── testinfra-expert.md
│   ├── alpine-container-expert.md
│   └── ubi-redhat-expert.md
├── commands/                    # 3 slash commands
│   ├── precommit.md
│   ├── save-todos.md
│   └── tox.md
├── hooks/
│   └── hooks.json               # PostToolUse + PreCompact hooks
├── CLAUDE.md                    # Engineering principles (always active)
├── README.md
└── LICENSE
```

## Requirements

Some agents require additional tools:

### Python Development

```bash
# Install uv for Python package management
curl -LsSf https://astral.sh/uv/install.sh | sh

# Install development tools
uv tool install ruff mypy pytest
```

### Ansible Development

```bash
# Install Ansible toolchain
uv tool install --with-executables-from ansible-core,ansible-lint,molecule \
    --with 'molecule-plugins[docker]',testinfra ansible
```

### Container Development

```bash
# Ensure Docker/Podman is installed and BuildKit is enabled
export DOCKER_BUILDKIT=1
```

## MCP Integration

This plugin works best with the Context7 MCP server for documentation lookups. Enable it in your Claude Code settings:

```json
{
  "enabledPlugins": {
    "context7@claude-plugins-official": true
  }
}
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Run pre-commit checks
5. Submit a pull request

## License

Apache License 2.0 - See [LICENSE](LICENSE) for details.

## Support

For issues and feature requests, please use the GitHub issue tracker.
