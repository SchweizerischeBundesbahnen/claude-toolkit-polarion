---
description: Save current TodoWrite state to TODO.md for cross-session persistence
---

Please save the current TodoWrite state to TODO.md with the following format:

```markdown
# TODO - Last updated: [current date/time]

## Current Tasks

[For each todo in TodoWrite, format as:]
- [ ] Task description (if pending)
- [x] Task description (if completed)
- [~] Task description (if in-progress)

## Notes
- This file is user-maintained and optional
- Survives /compact and session resets
- Update manually when needed for cross-session persistence
```

After saving, confirm the backup was created.
