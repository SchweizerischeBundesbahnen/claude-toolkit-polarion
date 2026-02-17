# Claude Toolkit Polarion

A monorepo marketplace providing specialized Claude Code plugins for Java, JavaScript/TypeScript, Python, Ansible, containers, Polarion, and development workflows.

## Available Plugins

| Plugin | Agents | Skills | Description |
|--------|--------|--------|-------------|
| **ansible** | 3 | — | Ansible infrastructure automation, Molecule testing, Testinfra verification |
| **containers** | 2 | — | Alpine Linux and Red Hat UBI container optimization |
| **core** | 2 | 4 | Documentation lookup, research analysis, workflow commands, hooks |
| **java** | 4 | 2 | Java 21+ development, Spring Boot, Maven, JUnit 5 |
| **javascript** | 4 | 2 | JavaScript/TypeScript, React, Node.js, Jest/Vitest |
| **polarion** | — | 1 | Local ALM Polarion service management |
| **python** | 4 | — | Python/FastAPI development, API design, debugging, Ruff linting |

## Installation

### Add the Marketplace

```bash
/plugin marketplace add SchweizerischeBundesbahnen/claude-toolkit-polarion

# Or pin to a specific version/branch:
/plugin marketplace add SchweizerischeBundesbahnen/claude-toolkit-polarion#v1.0.0
/plugin marketplace add SchweizerischeBundesbahnen/claude-toolkit-polarion#main
```

### Install Individual Plugins

```bash
# Install only what you need
/plugin install ansible@claude-toolkit-polarion
/plugin install containers@claude-toolkit-polarion
/plugin install core@claude-toolkit-polarion
/plugin install java@claude-toolkit-polarion
/plugin install javascript@claude-toolkit-polarion
/plugin install polarion@claude-toolkit-polarion
/plugin install python@claude-toolkit-polarion
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

### Containers Plugin

| Agent | Description |
|-------|-------------|
| **alpine-container-expert** | Alpine Linux optimization, musl compatibility, security |
| **ubi-redhat-expert** | Red Hat UBI/OpenShift containers, glibc compatibility |

### Core Plugin

| Agent | Description |
|-------|-------------|
| **context7-assistant** | Documentation assistant using Context7 MCP |
| **deep-research-analyst** | Internet research with critical analysis |


| Skill | Description |
|-------|-------------|
| `/commit` | Interactive git commit with ticket reference and pre-commit checks |
| `/precommit` | Run pre-commit checks on staged or all files |
| `/save-todos` | Persist TodoWrite state to TODO.md |
| `/tox` | Run tox test suite with environment options |

**Hooks:**
- PostToolUse: Reminds to run pre-commit after file modifications
- PreCompact: Saves todos before context compaction

### Java Plugin

| Agent | Description |
|-------|-------------|
| **java-developer** | Clean Java 21+ code, SOLID principles, modern features |
| **spring-boot-expert** | Spring Boot applications, REST APIs, JPA, Security |
| **maven-expert** | Maven build configuration, dependency management, multi-module projects |
| **java-test-expert** | JUnit 5, Mockito, AssertJ, Testcontainers |

| Skill | Description |
|-------|-------------|
| `/mvn` | Run Maven build commands |
| `/java-code-review` | Systematic Java code review |

### JavaScript Plugin

| Agent | Description |
|-------|-------------|
| **js-developer** | Modern JavaScript/TypeScript, ES modules, async/await |
| **react-expert** | React applications, functional components, hooks |
| **node-expert** | Node.js backend, Express/Fastify, npm packages |
| **js-test-expert** | Jest, Vitest, React Testing Library |

| Skill | Description |
|-------|-------------|
| `/npm-run` | Run npm scripts |
| `/js-code-review` | Systematic JS/TS code review |

### Polarion Plugin

| Skill | Description |
|-------|-------------|
| `/polarion-restart` | Restart local Polarion service with workspace cache clearing |

### Python Plugin

| Agent | Description |
|-------|-------------|
| **python-developer** | Clean Python, FastAPI, async patterns, PEP standards |
| **api-developer** | REST/GraphQL API design, OpenAPI documentation |
| **code-debugger** | Systematic debugging and root cause analysis |
| **ruff-expert** | Ruff linter configuration |

## Agent Design

Agents in this toolkit are intentionally minimal. Each agent covers only what Claude cannot discover itself: domain-specific hazards, non-obvious constraints, and project-specific landmines. General best practices and code templates are deliberately omitted — they add noise and anchor Claude to specific patterns even when inappropriate.

## Repository Structure

```
claude-toolkit-polarion/
├── .claude-plugin/
│   └── marketplace.json        # Marketplace manifest
├── plugins/
│   ├── ansible/
│   │   ├── .claude-plugin/plugin.json
│   │   └── agents/
│   ├── containers/
│   │   ├── .claude-plugin/plugin.json
│   │   └── agents/
│   ├── core/
│   │   ├── .claude-plugin/plugin.json
│   │   ├── agents/
│   │   ├── hooks/
│   │   └── skills/
│   ├── java/
│   │   ├── .claude-plugin/plugin.json
│   │   ├── agents/
│   │   └── skills/
│   ├── javascript/
│   │   ├── .claude-plugin/plugin.json
│   │   ├── agents/
│   │   └── skills/
│   ├── polarion/
│   │   ├── .claude-plugin/plugin.json
│   │   └── skills/
│   └── python/
│       ├── .claude-plugin/plugin.json
│       └── agents/
├── CLAUDE.md
└── README.md
```

## Requirements

### Context7 Plugin (for context7-assistant agent)

Several agents use [Context7](https://context7.com) for live documentation lookups. Install from the [official Anthropic marketplace](https://github.com/anthropics/claude-plugins-official):

```bash
/plugin install context7@claude-plugins-official
```

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
