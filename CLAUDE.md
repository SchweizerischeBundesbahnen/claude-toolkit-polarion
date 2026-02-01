# SBB Polarion Claude Toolkit - Configuration

## MCP Tool Usage Priority

**Use MCP tools proactively. General priority order (project-specific overrides apply):**

1. **Context7** - Library documentation
2. **Other MCPs** - As appropriate for the task

## Software Engineering Principles (CRITICAL)

**ALWAYS apply these principles when writing code:**

### DRY (Don't Repeat Yourself)

- **Configuration-driven design**: Use config files/objects instead of hardcoded values
- **Single source of truth**: Changes should require modification in only ONE place
- **Dynamic generation**: Tables, lists, and data structures should adapt automatically to configuration changes
- **Example**: Adding a new test case should only require updating a config array, not searching through 600+ lines of code

### Single Responsibility Principle

- **Each function/class has ONE clear purpose**
- **Separation of concerns**: Configuration loading, data processing, and output generation should be separate
- **Modular design**: Components should be easily replaceable and testable
- **Example**: A test runner should not hardcode test parameters - it should load them from configuration

### Configuration-First Design Pattern

- **External configuration**: All variable parameters in JSON/YAML files
- **Runtime flexibility**: Same code should handle different configurations without modification
- **Extensibility**: Adding new options should not require code changes
- **Maintainability**: Configuration changes are safe and don't introduce bugs

### Performance and Usage Measurement Principle (CRITICAL)

**NEVER SPECULATE - ALWAYS MEASURE**

- **Memory usage**: Measure actual memory consumption, never use hardcoded estimates
- **Performance metrics**: Measure actual response times, throughput, latency
- **Resource utilization**: Measure CPU, GPU, disk usage during actual operations
- **Model sizes**: Measure actual model memory footprint when loaded
- **System overhead**: Measure real system resource consumption
- **Examples of what to measure**: RAM usage before/after model loading, actual response times with timers, real disk space used, actual network bandwidth consumed
- **Never assume or estimate**: "It probably uses ~12GB" → measure actual usage
- **Measure in real conditions**: Test with actual workloads, not synthetic examples

### Test-Driven Development (TDD)

**WRITE TESTS FIRST, THEN IMPLEMENTATION**

**Core TDD Principles:**

- **Red-Green-Refactor cycle**: Write failing test → Make it pass → Refactor code
- **Test before code**: Define expected behavior through tests before writing implementation
- **Comprehensive coverage**: All code paths should be tested, including edge cases and error conditions
- **Regression prevention**: Tests catch breaking changes immediately
- **Integration with CI/CD**: All tests must pass before commits/merges

**CRITICAL: Project-Specific Testing Guidelines**

- **ALWAYS read `tests/README.md` before writing or running tests**
- The tests/README.md file contains:
  - Test structure and organization
  - Test execution commands and workflows
  - Test design patterns and best practices
  - Framework-specific guidelines
  - Coverage requirements
  - CI/CD integration details
- **If tests/README.md doesn't exist**, ask user if they want to create it with testing guidelines
- Project-specific test documentation OVERRIDES these general TDD principles

### Spec-Driven Development (SDD)

**WRITE SPECIFICATIONS FIRST, THEN PLAN AND IMPLEMENT**

**Core Workflow:**

- Specifications are the **single source of truth** (living markdown in `.github/spec.md`)
- **SDD provides "what"** (requirements, architecture, contracts) → **TDD provides "how"** (tests, implementation)
- Flow: Write spec.md → Break into TODO.md tasks → Implement with TDD → Update spec

**CRITICAL: Read .github/spec.md First**

- **ALWAYS read `.github/spec.md` before starting implementation**
- If doesn't exist, ask user if they want to create it
- Project-specific specifications OVERRIDE these general principles

**Recommended spec.md Structure:**

```markdown
# Specification: [Feature Name]
## Problem Statement
## Solution Overview
## Architecture
## API/Interface Contract
## Implementation Plan (high-level milestones - daily tasks go in TODO.md)
## Testing Strategy (links to tests/README.md)
## Open Questions
```

**Spec → TODO → Implementation:**

- **.github/spec.md** = Strategic (architecture, milestones, contracts)
- **TODO.md** = Tactical (daily tasks from spec milestones)
- **Code** = Implementation (fulfills spec + completes TODOs)
- Update both as work progresses

## Response Guidelines

- Provide thorough responses when needed, but remain concise when appropriate
- Minimize output tokens while maintaining quality
- Answer directly without unnecessary preambles
- Use tools proactively without asking permission
- Focus on the specific query/task at hand

## Code Review Protocol

When asked to review code/changes/files:

**Focus on:**

- Bugs or logic errors
- Security vulnerabilities (SQL injection, XSS, auth bypass, secrets exposure)
- Breaking changes (API changes, removed functionality)
- Missing tests for new functionality
- Critical performance issues introduced by changes

**DO NOT review:** unchanged code, style/formatting, optional improvements, or provide praise.

**Be terse.** Format: `file:line` - Problem - Fix: solution

## Security and Privacy

- **NEVER expose internal information** in any output, code, documentation, or external systems (GitHub issues, etc.):
  - Internal URLs (e.g., *.sbb.ch domains except public ones like www.sbb.ch)
  - Internal server names, hostnames, or network configurations
  - Internal email addresses (use example.com instead)
  - Internal project identifiers or ticket numbers
  - Internal employee names or usernames
  - Internal infrastructure details
- Use generic examples: example.com, user@example.com, PROJECT-123, localhost

## Development Practices

- Check IDE diagnostics for errors (use mcp__ide__getDiagnostics if available)

### CRITICAL: Context Window Strategy (MANDATORY)

**PRIMARY GOAL: PREVENT CONTEXT WINDOW EXHAUSTION - ALWAYS USE SUBAGENTS**

**MUST use agents (Task tool) for:**

- Research, documentation lookups, web searches
- Codebase exploration ("where is X?", "how does Y work?")
- Code reviews and debugging investigations
- Any task requiring 3+ file reads
- Multi-step analysis tasks

**Main context ONLY for:**

- Direct edits to specific known files
- Simple 1-2 file operations
- Running commands/tests

**Agent spawning time is ALWAYS acceptable - never fill main context!**

### Agent Philosophy

**Core Philosophy (Per Official Claude Code Docs):**
Create subagents with **single, clear responsibilities** for focused tasks. Agents preserve main context, enable expert task handling, and support reusable workflows across projects.

**When to Create Agents:**

1. **Domain-specific expertise** - Specialized knowledge (debugging, code review, Ansible, containers)
2. **Reusable workflows** - Consistent patterns used across multiple projects
3. **Context separation** - Tasks needing separate context to avoid polluting main conversation
4. **Tool restriction** - Workflows requiring limited tool access for security/focus

**PROACTIVELY Pattern:**
Agents with "Use PROACTIVELY" in their description are automatically invoked for their domain.

### Other Development Practices

- **Pre-commit is MANDATORY** before committing - never skip it
  - PostToolUse hook reminds after Edit/Write operations
  - Run manually with `/precommit` command
- NEVER suppress or comment lint or test errors or problems
- Use .ssh/config for SSH connections

## CRITICAL: Task Tracking (MANDATORY)

**TodoWrite-First with Semi-Automated Persistence**

**ALWAYS use TodoWrite as the primary task tracking tool**

- Use TodoWrite for its native UI integration and structured format
- Provides clean task management with proper state tracking (pending → in-progress → completed)
- Zero risk of git pollution
- System provides helpful reminders for multi-step tasks

**TodoWrite Workflow:**

1. For multi-step tasks, create todos using TodoWrite tool
2. Update todo status as work progresses (mark in-progress, then completed)
3. Complete tasks and close them out properly

**TODO.md for Cross-Session Persistence**

- **Automatically saved before `/compact`** via PreCompact hook
  - Hook outputs DIRECTIVE that Claude acts on immediately
  - Ensures TodoWrite state survives compaction
- **Manual save anytime** with `/save-todos` command
- **Format:** Markdown with checkboxes (`[ ]` pending, `[~]` in-progress, `[x]` completed)
- **Git handling** - Already excluded by whitelist `.gitignore`. To commit it later, add `!TODO.md` to `.gitignore`

**Relationship to .github/spec.md (SDD):**

- **spec.md** = Strategic plan (high-level milestones, architecture decisions)
- **TODO.md** = Tactical implementation (day-to-day tasks derived from spec milestones)
- **Workflow**: Spec milestones → Break into TODO.md tasks → Track progress → Update both
- **Example**: spec.md says "Implement user authentication", TODO.md has "[ ] Create User model", "[ ] Add login endpoint", etc.

**When TODO.md exists:**

- Claude reads it at session start to restore context after `/compact`
- Provides visibility in IDE file explorer for reference
- Can be manually edited if needed
- Should reference spec.md sections when implementing strategic features

## Documentation Organization

**Keep documentation properly separated by purpose:**

**CLAUDE.md (Project guidance for AI):**

- Project architecture and structure
- Development commands and workflows
- Configuration guidance
- Best practices and conventions
- NOT for progress tracking or coverage statistics

**TODO.md (Tactical task tracking - semi-automated):**

- Automatically created before `/compact` via PreCompact hook
- Manual creation/updates via `/save-todos` command
- Day-to-day implementation tasks derived from .github/spec.md milestones
- Progress tracking (checkboxes, completion status)
- Coverage statistics and test counts
- Implementation details and notes
- Survives context compaction and session resets

**README.md (User-facing documentation - committed):**

- Installation instructions
- Quick start examples
- API usage examples
- NOT for internal development details

**If you add progress/coverage info to CLAUDE.md, move it to TODO.md immediately.**

## Documentation Maintenance

- ALWAYS update project README.md after implementing features
- Keep CLAUDE.md focused on guidance, not progress tracking
- Use TODO.md for work-in-progress notes and coverage stats
