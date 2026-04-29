---
name: Verify rule IDs and file counts at HEAD before proposing skill changes
description: Before relocating, renaming, or adding rule IDs in skill CSVs, grep the canonical CSV files at HEAD. Don't propose IDs that already exist or count files from earlier-session synthesis.
type: feedback
originSessionId: e5297319-4236-4e52-b33e-eeafef657594
---
When proposing changes to `.claude/skills/**/data/rules/*.csv`, every rule ID in the proposal must be verified against the canonical CSV at HEAD before the plan is finalised.

**Why:** On 2026-04-27 a widget-test-first-class plan proposed:
- Relocating `GOTCHA-TEST-003` to `INFRA-COMP-002` — but `INFRA-COMP-002` was already occupied by the PinnedFlutter golden-CI rule (`infra_tests.csv:9`). The collision would have silently overwritten the existing rule on apply.
- Adding `WIDGET-API-PUMPFRAMES-001` and `WIDGET-TIME-FAKEASYNC-001` — but `flaky_tests.csv:2-3` already had `FLAKY-PUMP-001` (mandates `pump(Duration)` over `pumpAndSettle` for infinite animations) and `FLAKY-TIME-001` (mandates `Clock.fixed`). The new rules duplicated existing coverage.
- "7 cockpit cells without tests" — but the cockpit has **8 cells**, not 7. `graph_playground_cell.dart` is the 8th cell, already covered by `apps/tutorials_admin/test/widget/graph_playground_lifecycle_test.dart`. The earlier exploration synthesis missed this.
- `WIDGET-DIALOG-001` proposed as a new rule — but `scripts/decision-trees/test_type_decision.dart:215` already emitted that ID against a missing CSV row. It was an orphan-resolution backfill, not a new rule.

The user caught all four in audit and rejected the locked plan with the verdict "would not implement as-is."

**How to apply:**

1. **Before proposing any rule ID** (new, relocated, or renamed): run
   `grep -rn "<ID-PATTERN>" .claude/skills/**/data/rules/*.csv`
   to confirm uniqueness and discover near-collisions.
2. **Before proposing new pump/time/lifecycle/state discipline rules**: grep `FLAKY-*` family in `flaky_tests.csv` and `TASTE-*` in `taste_meta_rules.csv` and `CHAIN-*` in `chain_tests.csv`. These cross-cutting families already cover most "test discipline" territory.
3. **Before counting files** (e.g. "7 untested cells"): run `ls` / `find` at HEAD against the named directory. Do not count from session synthesis or prior plan documents. Counts decay between sessions.
4. **Before proposing a new linter or coverage gate**: distinguish the two concerns explicitly. A rule-reference linter (validates rule IDs resolve to canonical CSV entries) is NOT the same as a coverage checker (validates a source file has a paired test file). The user's reviews flag conflation.
5. **Before proposing rule renames** (e.g. `WIDGET-MANDATE-001` → `WIDGET-SURFACE-MANDATE-001`): explicitly scope what the rule applies to (rendered surfaces vs all source files). Vague mandate rules generate disputes during enforcement.
6. **Before proposing a remediation that edits generated files** (e.g. "comment out SupadartClient extensions with `/* */`"): treat the manual edit as a smell. The fix is usually a codegen/dependency configuration change, not a manual generated-file edit.
