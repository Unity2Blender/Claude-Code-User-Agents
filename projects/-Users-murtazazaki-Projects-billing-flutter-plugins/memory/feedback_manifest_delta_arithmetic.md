---
name: test/manifest.json deltas need arithmetic correctness or check-rule-refs breaks
description: When adding a module entry to test/manifest.json, both the module's testCount and the summary block must be recomputed; path must point at the actual changed source dir.
type: feedback
originSessionId: 1cab8526-61e0-4cc7-81dd-a06e3f52e4fc
---
`test/manifest.json` deltas require three pieces of arithmetic that are easy to get wrong:

1. **Module `testCount`** must equal `sum(target.testCount)` across all targets in the module. Setting it to 0 while targets total 12 (e.g. 10 + 1 + 1) breaks `make check-rule-refs` and the testing-DX flywheel coverage map.
2. **Module `path`** must point at the real source directory the targets exercise. A `paywall_gate` module whose targets cover `apps/gst_calculator/lib/utils/billing_action_gate.dart` should have `path: "apps/gst_calculator/lib/utils/"`, not the services directory.
3. **`summary` block** must be recomputed BOTH for `totalModules` (+1 per new module) AND `testedModules` (+1 if new module is `status: "tested"`). `totalTargets`, `testedTargets`, `totalTestCount`, `coveragePercent` all recompute. Skipping `testedModules` is a common miss because it's not visually adjacent to `totalModules`.

**Rule.** When editing `test/manifest.json`:
- Compute module `testCount` as the literal sum of target `testCount` values; verify by hand.
- Choose `path` based on what the targets test, not where the test files live.
- Update `summary` for both `totalModules` AND `testedModules`. Recompute `coveragePercent = round(testedTargets / totalTargets * 100, 1)`.
- After save, run `make check-rule-refs` AND `make check-widget-surfaces` (especially when deleting rendered surfaces) to catch broken rule references and surface-mandate violations.

**Why:** 2026-05 audit on the paywall-teaser-shown plan flagged `testCount: 0` with 12 target tests, missing `testedModules` increment, and a `path` pointing at `services/` when targets covered `utils/`. All three are silent until `make check-rule-refs` fails in CI; correcting at audit time is cheaper than fixing in CI.

**How to apply:** Use this as a checklist before committing manifest changes:
- [ ] `module.testCount == sum(targets[].testCount)`
- [ ] `module.path` points at source dir, not test dir
- [ ] `summary.totalModules` incremented
- [ ] `summary.testedModules` incremented if `status: 'tested'`
- [ ] `summary.totalTargets`, `testedTargets`, `totalTestCount` updated
- [ ] `summary.coveragePercent` recomputed
- [ ] Ran `make check-rule-refs` post-edit
- [ ] Ran `make check-widget-surfaces` post-edit (if rendered surfaces changed)
