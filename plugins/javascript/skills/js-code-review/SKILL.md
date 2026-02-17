---
name: js-code-review
description: Systematic code review for JavaScript/TypeScript with modern pattern enforcement, error handling, and React checks. Use when reviewing JS/TS code or before merging changes.
allowed-tools: Read, Glob, Grep
---

# JavaScript/TypeScript Code Review

Perform a systematic review of JS/TS code changes, focusing on modern patterns, correctness, and maintainability.

## Current Context

**Changed JS/TS files:**
`!`git diff --name-only HEAD~1 -- '*.js' '*.ts' '*.jsx' '*.tsx' 2>/dev/null || git diff --cached --name-only -- '*.js' '*.ts' '*.jsx' '*.tsx' 2>/dev/null || echo "No JS/TS changes found"``

**Unstaged JS/TS changes:**
`!`git diff --name-only -- '*.js' '*.ts' '*.jsx' '*.tsx' 2>/dev/null || echo "No unstaged JS/TS changes"``

## Review Checklist

Review each changed file against these categories:

### 1. Modern JS Enforcement
- Are ES modules used (`import`/`export`)? Flag any `require()` or `module.exports`.
- Is `async`/`await` used? Flag `.then()` chains and callback patterns.
- Is `const`/`let` used? Flag any `var` declarations.
- Are optional chaining (`?.`) and nullish coalescing (`??`) used where appropriate?

### 2. Error Handling
- Are async functions wrapped in try/catch?
- Are errors propagated correctly (not swallowed silently)?
- Are custom error classes used for domain errors?
- Are unhandled promise rejections possible?

### 3. TypeScript Quality (if applicable)
- Is `any` used? Flag it — use `unknown` with type narrowing instead.
- Are function parameters and return types explicitly typed?
- Are `interface`/`type` definitions used for object shapes?
- Is `strict: true` enabled in tsconfig.json?

### 4. React Patterns (if applicable)
- Are class components used? Flag them — use functional components.
- Are hooks used correctly (dependencies array, cleanup)?
- Is `useEffect` used for derived state? Flag it — compute during render.
- Are array indices used as `key` prop? Flag for dynamic lists.
- Are components created inside other components? Flag it.

### 5. Security
- Are user inputs sanitized before use?
- Are secrets hardcoded? Flag any API keys, tokens, passwords.
- Is `eval()` or `Function()` constructor used? Flag it.
- Is `dangerouslySetInnerHTML` used without sanitization?

### 6. Performance
- Are expensive computations wrapped in `useMemo`/`useCallback` where needed?
- Are large lists virtualized?
- Is unnecessary re-rendering happening (missing React.memo, unstable references)?
- Are N+1 API calls present (fetch in a loop)?

## Output Format

Report findings as:

```
file:line - [CATEGORY] Problem description - Fix: suggested solution
```

**Severity levels:**
- **BUG** - Will cause incorrect behavior
- **SECURITY** - Potential vulnerability
- **PERFORMANCE** - Significant performance impact
- **IMPROVEMENT** - Code quality suggestion

Only report actual issues. Do not comment on unchanged code, style preferences, or provide praise.
