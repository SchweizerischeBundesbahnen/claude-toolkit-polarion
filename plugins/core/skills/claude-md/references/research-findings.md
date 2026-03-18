# Research Findings: AGENTS.md / CLAUDE.md Context Files

Source: https://sulat.com/p/agents-md-hurting-you

## Key Studies

### Gloaguen et al. (Feb 2026) — ETH Zurich AGENTBENCH
- LLM-generated context files **reduced performance in 5/8 experimental settings**
- Average decline: ~2% with LLM-generated files
- Human-written files: inconsistent — ~4% average improvement but varied by agent
- **Claude Code specifically performed worse with human-written context** than with no file
- Context files increased inference costs **20–23%**

### Lulla et al. (Jan 2026) — Efficiency Analysis
- Files improved agent **efficiency** (faster navigation, fewer tool calls)
- Files did NOT improve **task success rates**
- Agents spent **14–22% more reasoning tokens** when given context files
- File discovery rates were identical with or without context — agents find files on their own

### Liu et al. — U-Shaped Attention in LLMs
- LLMs exhibit "U-shaped attention": strong focus on beginning and end of long inputs
- Middle sections of long CLAUDE.md files are largely ignored
- Implication: long files waste tokens and dilute the impact of critical facts

## Key Anti-Patterns Found in the Wild

- **100% of Sonnet 4.5-generated CLAUDE.md files** contained redundant codebase overviews
- Agents **over-anchor on file mentions**: tool usage jumped 1.6–160x based on what was mentioned in context, even inappropriately

## Emerging Alternatives (for reference)

These approaches show promise but are more complex to implement:

- **ACE Framework**: Task-specific dynamic context generation — 12.3% performance improvement
- **Arize AI approach**: Iterative refinement loop with agent feedback — 5–11% accuracy gains
- **Layered architecture**: Protocol routing to focused persona files with automated maintenance

## Practical Synthesis (Addy Osmani, Google)

Filter context files aggressively. The only content worth including is information that:
1. An agent cannot derive from reading the code
2. Would cause a failure or significant wasted effort if unknown
3. Is not documented elsewhere (README, config files, CI)
