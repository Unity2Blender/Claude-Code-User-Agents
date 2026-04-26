---
name: Canonical single-sequence plans — no superseded layers
description: Plans must be one canonical sequence; never accumulate original + audit-amendments as parallel layers
type: feedback
originSessionId: 12cb5d17-5a59-4353-be5f-3343fd691c0d
---
When a plan goes through Ultraplan refinement / audit passes, fold every accepted amendment directly into the canonical directive list. Do NOT append an "Audit Amendments" section that overrides earlier directives — that creates internal contradictions (directive X says one thing, appendix says another) and makes the plan unreviewable.

**Why:** User rejected ExitPlanMode on 2026-04-18 for the tutorials CTA/reels plan because:
- Phase order contradictions (D27 "extends D15" but D15 was Phase 2 while D27 was Phase 1.5)
- Launch gate listed Phase 2 items as required — making "Phase 2 enhancements" effectively launch-blocking
- Duplicate directives (new + amended) both appeared in the file
- Too many parallel mirrors of CTA truth without clear SSOT ownership

**How to apply:**
- After each audit pass, REWRITE the plan file (not Edit-patch) so amendments are absorbed inline into each directive's definition
- Remove superseded text entirely — don't leave it crossed out or in an "AMEND-N overrides Dx" format
- Launch gate exit criteria must match the actual Phase 1 scope (not reference later phases)
- One SSOT for any enum-like value; all other mirrors are projections/tests, documented as such
- Phase ordering: if directive B depends on directive A, B must be in the same or later phase — verify with a dependency pass before claiming "ready"
- Feature flags and min_build_number gate out anything not strictly required for launch
