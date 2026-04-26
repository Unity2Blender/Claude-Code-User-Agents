---
name: Don't over-complicate plans — focus on the user's stated pain point
description: When the user describes a specific blocker, scope the plan tightly. Don't bolt on speculative re-baselining tooling, schemas/-survival debates, or future-proofing.
type: feedback
originSessionId: 83fb8343-bb17-4517-a0c9-061d55c22211
---
When the user describes a concrete pain point ("local Docker is broken on migration 12, prod is fine"), focus the plan tightly on that goal. Don't add speculative future-proofing or brainstorming branches.

**Why:** During the v3 rebaseline planning, my first draft added a `make rebaseline` repeatable-tooling deliverable, schemas/-survival debate, and 12-gotcha audit BEFORE the user even confirmed the basic shape. User pushback: *"You're making too much complications and too much brainstorming cases. The only thing I was concerned about was that the prod is fine, local Docker is not."*

**How to apply:**
- Start the plan with one sentence stating the user's exact pain point and goal.
- Lock decisions only on what the pain point requires; defer "make this not painful next time" wrappers to Post-Implementation Follow-ups (or skip entirely until the user asks).
- Surface only the load-bearing audit items that map to specific failure modes. Don't itemize every gotcha from the rule index.
- When the user says "review and audit for shortcomings," that's permission to add depth — but still scope each finding to the user's pain point, not adjacent concerns.
- AskUserQuestion calls should resolve true architectural choices (D2/D3-style), not hypothetical edge cases.
