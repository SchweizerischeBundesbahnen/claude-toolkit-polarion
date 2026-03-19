---
name: triage-code-reviews
description: Use this skill when the user asks to triage, review, or address review comments on a PR (from bots or humans). Triggers when user asks to "go through reviews", "handle review comments", "triage the PR", "address feedback", or similar.
version: 1.0.0
---

Triage all open review comments on the current PR. Follow this workflow:

1. Fetch all open review threads
2. Assess each comment independently: real bug, valid improvement, or false positive/nitpick?
3. Present a triage table — one row per comment with your verdict (Fix / Dismiss) and a one-line reason
4. Discuss with the user — answer questions, accept corrections, adjust verdicts based on their input
5. Once the discussion is settled, apply the fixes and commit them
6. At the very end, ask: "Do you want me to reply to and resolve the bot comments on GitHub?"
   - If yes: post a dismissal reply on dismissed threads, then resolve all threads via GraphQL

Do NOT fix anything before the discussion is complete. Think critically — do not apply changes just to clear a thread.

Use `references/github-api.md` for the correct API calls to reply to and resolve threads.

Start by fetching the open review threads for the current PR.
