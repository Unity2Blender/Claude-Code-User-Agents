---
description: Format a prompt for ralph-wiggum loop execution
---

Output exactly this command with the user's prompt inserted:

/ralph-wiggum:ralph-loop '$ARGUMENTS' --completion-promise "DONE" --max-iterations 10 --dangerously-skip-permissions

Use single quotes around the prompt to avoid bash literal issues with double quotes.
