# Engineering Principles Skill

This skill provides software engineering principles and best practices guidance for all development work.

## DRY (Don't Repeat Yourself)

- **Configuration-driven design**: Use config files/objects instead of hardcoded values
- **Single source of truth**: Changes should require modification in only ONE place
- **Dynamic generation**: Tables, lists, and data structures should adapt automatically to configuration changes
- **Example**: Adding a new test case should only require updating a config array, not searching through 600+ lines of code

## Single Responsibility Principle

- **Each function/class has ONE clear purpose**
- **Separation of concerns**: Configuration loading, data processing, and output generation should be separate
- **Modular design**: Components should be easily replaceable and testable
- **Example**: A test runner should not hardcode test parameters - it should load them from configuration

## Configuration-First Design Pattern

- **External configuration**: All variable parameters in JSON/YAML files
- **Runtime flexibility**: Same code should handle different configurations without modification
- **Extensibility**: Adding new options should not require code changes
- **Maintainability**: Configuration changes are safe and don't introduce bugs

## Performance and Usage Measurement Principle (CRITICAL)

**NEVER SPECULATE - ALWAYS MEASURE**

- **Memory usage**: Measure actual memory consumption, never use hardcoded estimates
- **Performance metrics**: Measure actual response times, throughput, latency
- **Resource utilization**: Measure CPU, GPU, disk usage during actual operations
- **Model sizes**: Measure actual model memory footprint when loaded
- **System overhead**: Measure real system resource consumption
- **Examples of what to measure**: RAM usage before/after model loading, actual response times with timers, real disk space used, actual network bandwidth consumed
- **Never assume or estimate**: "It probably uses ~12GB" → measure actual usage
- **Measure in real conditions**: Test with actual workloads, not synthetic examples

## Test-Driven Development (TDD)

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

## Spec-Driven Development (SDD)

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

## Code Quality Standards

### Code Review Focus
When reviewing code/changes/files:

**Focus on:**
- Bugs or logic errors
- Security vulnerabilities (SQL injection, XSS, auth bypass, secrets exposure)
- Breaking changes (API changes, removed functionality)
- Missing tests for new functionality
- Critical performance issues introduced by changes

**DO NOT review:** unchanged code, style/formatting, optional improvements, or provide praise.

**Be terse.** Format: `file:line` - Problem - Fix: solution

### Security and Privacy
- **NEVER expose internal information** in any output, code, documentation, or external systems
- Use generic examples: example.com, user@example.com, PROJECT-123, localhost

### Development Practices
- Check IDE diagnostics for errors
- Pre-commit is MANDATORY before committing - never skip it
- NEVER suppress or comment lint or test errors or problems
