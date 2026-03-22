---
name: commit
description: Interactive git commit with ticket reference and pre-commit checks. Enforces conventional commits + "Refs: #XXX" format.
allowed-tools: Bash(git *), Bash(uv *), Bash(pre-commit *), AskUserQuestion
---

# Interactive Git Commit

Guides you through creating a git commit with conventional commit format and ticket references.

## Current Context

**Git Status:**
`!`git status --short``

**Staged Changes:**
`!`git diff --cached --stat``

**Recent Commits (for reference):**
`!`git log --oneline -5``

## Workflow

### 1. Verify Staged Changes

Check if there are staged changes. If no changes are staged:
1. Show unstaged files with `git status --short`
2. Use AskUserQuestion to let the user pick which files to stage
3. Stage only the selected files by name — **never use `git add .` or `git add -A`**

### 2. Collect Commit Information

Use AskUserQuestion to collect:

**Question 1: Commit Type**
- header: "Type"
- options: feat, fix, docs, chore, refactor, test, ci, perf, style, build
- Include descriptions for each type

**Question 2: Commit Message**
- header: "Message"
- Prompt: "Enter the commit message (brief description of changes):"
- This will be a free-form text input

**Question 3: Ticket Number** (conditional)
- header: "Ticket"
- Prompt: "Enter the ticket/issue number (e.g., 136) or 'skip':"
- Only required if type is NOT "chore"
- For "chore" commits, skip this question automatically

### 3. Run Pre-commit Checks

**MANDATORY:** Run pre-commit hooks before committing:
```bash
uv run pre-commit run
```

If pre-commit fails:
1. Display failures clearly
2. Fix issues automatically when possible
3. Re-run pre-commit to verify
4. DO NOT proceed with commit until all checks pass

### 4. Format and Execute Commit

Format the commit message based on collected information:

**With ticket number:**
```
<type>: <message>

Refs: #<ticket>
```

**Without ticket number (chore only):**
```
chore: <message>
```

Execute the commit using HEREDOC format:
```bash
git commit -m "$(cat <<'EOF'
<type>: <message>

Refs: #<ticket>
EOF
)"
```

### 5. Verify Commit

After committing, show the result:
```bash
git log -1 --pretty=fuller
git status
```

## Example Usage

**Feature commit with ticket:**
```
feat: add user authentication

Refs: #136
```

**Chore commit without ticket:**
```
chore: update dependencies
```

## Notes

- Pre-commit hooks are MANDATORY and will run automatically
- Ticket numbers are required for all commit types except "chore"
- Follows conventional commit specification
- Commit message format is enforced by commitizen pre-commit hook
