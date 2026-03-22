---
name: mvn
description: Run Maven build commands. Use for building, testing, and managing Java projects.
allowed-tools: Bash(mvn *), Bash(./mvnw *), Bash(mvnw.cmd *)
---

# Maven Build

Run Maven to build, test, or manage the Java project.

## Current Context

**Build files present:**
`!`ls pom.xml mvnw 2>/dev/null || echo "No Maven configuration found"``

## Execution Options

**Basic execution:**
```bash
mvn clean verify
```

**Common options:**

| Option | Command | Description |
|--------|---------|-------------|
| Clean build | `mvn clean verify` | Full clean build with tests |
| Tests only | `mvn test` | Run unit tests |
| Integration tests | `mvn verify` | Run unit + integration tests |
| Skip tests | `mvn package -DskipTests` | Build without running tests |
| Dependency tree | `mvn dependency:tree` | Show dependency hierarchy |
| Check updates | `mvn versions:display-dependency-updates` | Find outdated dependencies |
| Effective POM | `mvn help:effective-pom` | Show resolved POM |
| Single module | `mvn test -pl module-name` | Test specific module |

**Combining options:**
```bash
mvn clean verify -pl module-name     # Clean build specific module
mvn test -Dtest=MyTest               # Run specific test class
mvn test -Dtest=MyTest#methodName    # Run specific test method
```

## Output Handling

- Display build results clearly
- On failure, show the failing module, test, or compilation error
- Summarize pass/fail counts for test execution

## Fallback

If Maven Wrapper is available, prefer it:
```bash
# Linux/macOS
./mvnw clean verify

# Windows
mvnw.cmd clean verify
```
