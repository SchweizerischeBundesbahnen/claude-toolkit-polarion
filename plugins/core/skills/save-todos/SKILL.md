---
name: save-todos
description: Save TodoWrite state to TODO.md for cross-session persistence. Use before compaction or for backup.
allowed-tools: Read, Write
---

# Save TodoWrite State

Persist the current TodoWrite task list to TODO.md for cross-session recovery.

## TODO.md Format

Create or update TODO.md in the project root with this format:

```markdown
# TODO - Last updated: [YYYY-MM-DD HH:MM]

## Current Tasks

- [ ] Pending task description
- [~] In-progress task description
- [x] Completed task description

## Notes
- This file survives /compact and session resets
- Manually editable for cross-session persistence
- Reference .github/spec.md sections when implementing strategic features
```

## Execution Steps

1. Read current TodoWrite state (if available)
2. Read existing TODO.md (if it exists) to preserve any manual notes
3. Write TODO.md with:
   - Current timestamp
   - All tasks from TodoWrite with appropriate checkbox status
   - Preserved notes section
4. Confirm the backup was created

## Status Mapping

| TodoWrite Status | Checkbox |
|-----------------|----------|
| pending         | `[ ]`    |
| in_progress     | `[~]`    |
| completed       | `[x]`    |

## Integration

This skill is automatically triggered by the PreCompact hook to preserve task state before context compaction.
