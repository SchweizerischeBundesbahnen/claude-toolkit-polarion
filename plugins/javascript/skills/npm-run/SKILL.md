---
name: npm-run
description: Run npm scripts for building, testing, and managing JavaScript/TypeScript projects.
allowed-tools: Bash(npm *)
---

# npm Scripts

Run npm commands to build, test, or manage the project.

## Current Context

**Package files present:**
`!`ls package.json package-lock.json 2>/dev/null || echo "No npm configuration found"``

**Available scripts:**
`!`node -e "try { const p = require('./package.json'); console.log(Object.keys(p.scripts || {}).join(', ')); } catch(e) { console.log('No scripts found'); }" 2>/dev/null || echo "Could not read package.json"``

## Execution Options

**Basic execution:**
```bash
npm test
```

**Common options:**

| Option | Command | Description |
|--------|---------|-------------|
| Install deps | `npm ci` | Clean install from lock file (CI-safe) |
| Install deps | `npm install` | Install and update lock file |
| Run tests | `npm test` | Run test suite |
| Run build | `npm run build` | Build the project |
| Run linter | `npm run lint` | Run ESLint |
| Fix lint | `npm run lint -- --fix` | Auto-fix lint issues |
| Run script | `npm run <script>` | Run any package.json script |
| Check outdated | `npm outdated` | Show outdated dependencies |
| Audit | `npm audit` | Check for security vulnerabilities |

**Combining options:**
```bash
npm run lint && npm test            # Lint then test
npm test -- --coverage              # Test with coverage report
npm test -- --testPathPattern=utils # Test specific folder
```

## Output Handling

- Display script output clearly
- On failure, show the error message and exit code
- Summarize test pass/fail counts when running tests

## Fallback

If `package-lock.json` is missing, use `npm install` instead of `npm ci`.
