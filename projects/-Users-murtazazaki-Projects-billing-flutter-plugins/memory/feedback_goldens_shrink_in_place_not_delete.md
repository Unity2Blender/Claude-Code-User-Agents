---
name: Goldens — shrink in place, never delete-and-postpone
description: When goldens go stale due to a visual change, shrink the matrix to canonical N cases + regenerate in place + manual diff review in PR. Deletion + postpone is anti-pattern.
type: feedback
originSessionId: 36039f2b-32bf-4fe5-849f-bc2739e79e2d
---
When a planned visual change will invalidate existing golden snapshots, the right move is:

1. **Shrink the matrix** to a canonical N-case set (e.g., 30 → 8) covering the highest-signal permutations only.
2. **Regenerate in place** in the same PR via `flutter test --update-goldens` for the shrunk matrix.
3. **Manual visual diff review** of every PNG in the PR description before merge.

**Anti-pattern (do NOT do):** Delete the goldens entirely and "regenerate in a follow-up PR". This:
- Kicks the can — the follow-up rarely happens
- Loses the visual baseline — future regressions go undetected
- Confuses CI — the suite passes (no goldens = no failures) but you've silently disabled the safety net

**Why:** Goldens are the only artifact that locks "the rendered pixels look right". Deleting them creates a no-coverage zone. Shrinking + regenerating preserves the safety net at lower maintenance cost.

**How to apply:**
- When planning a visual change that will break N goldens, decide BEFORE the PR which cases stay (the canonical set) and which go (over-coverage).
- Reduce by axis: drop redundant scaler variations (Glados covers them), drop edge-case modes (covered by unit tests), keep the 2×2×2 hero/tab × light/dark × Mode A/B grid.
- In the PR, use `--update-goldens` for the shrunk set, then manually review every PNG diff in the PR description.
- Reviewer's job: confirm the visual change is intentional and matches the design spec.

The plan that re-locked this discipline: `image-1-docs-runbooks-lib-billbook-tuto-ethereal-sparrow` (2026-04-22, F9 + D11). 30 banner goldens shrunk to canonical 8 (hero/tab × light/dark × Mode A/B at scaler 1.0).
