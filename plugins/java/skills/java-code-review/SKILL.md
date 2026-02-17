---
name: java-code-review
description: Systematic code review for Java with null safety, exception handling, concurrency, and performance checks. Use when reviewing Java code or before merging changes.
allowed-tools: Read, Glob, Grep
---

# Java Code Review

Perform a systematic review of Java code changes, focusing on correctness, safety, and maintainability.

## Current Context

**Changed Java files:**
`!`git diff --name-only HEAD~1 -- '*.java' 2>/dev/null || git diff --cached --name-only -- '*.java' 2>/dev/null || echo "No Java changes found"``

**Unstaged Java changes:**
`!`git diff --name-only -- '*.java' 2>/dev/null || echo "No unstaged Java changes"``

## Review Checklist

Review each changed Java file against these categories:

### 1. Null Safety
- Are return types using `Optional` where appropriate?
- Are `@NonNull`/`@Nullable` annotations used consistently?
- Are null checks present before dereferencing?
- Are collections initialized (not left null)?

### 2. Exception Handling
- Are specific exceptions caught (not `Exception` or `Throwable`)?
- Are catch blocks non-empty (no swallowed exceptions)?
- Are resources closed with try-with-resources?
- Are custom exceptions used for domain errors?

### 3. Concurrency
- Is shared mutable state properly synchronized?
- Are collections thread-safe where needed (`ConcurrentHashMap`, etc.)?
- Are `@Transactional` boundaries correct?
- Are there potential race conditions?

### 4. Resource Management
- Are `AutoCloseable` resources in try-with-resources?
- Are database connections properly returned to pool?
- Are streams closed after use?
- Are file handles released?

### 5. API Design
- Are DTOs used instead of exposing entities?
- Is input validated with Bean Validation?
- Are HTTP status codes correct?
- Is the API backwards-compatible?

### 6. Type Safety — No `var`/`val`
- Is `var` (Java local variable type inference) or Lombok's `var`/`val` used? Flag it — explicit type declarations are required. Use concrete types instead.

### 7. Regex Safety
- Is `Pattern.compile()` called inside a loop or frequently-called method? Flag it — extract to a `static final Pattern` field.
- Does the regex contain nested quantifiers (e.g., `(a+)+`, `(.*a){10}`)? Flag as potential ReDoS — catastrophic backtracking can hang the JVM on malicious input.
- Is `.` used to match multi-line content without `Pattern.DOTALL`? Flag if the intent is to match across newlines.
- Do NOT flag regexes just for existing. Only flag when one of the above concrete issues applies.

### 8. Performance
- Are N+1 query problems present (JPA lazy loading)?
- Are large collections streamed instead of loaded into memory?
- Is String concatenation in loops using StringBuilder?
- Are unnecessary object allocations present?

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
