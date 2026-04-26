---
name: No-consumer code deletes today, never sunsets
description: Dead code with zero consumer at PR time = delete in same PR. Sunset clocks (30/60/90-day) are for production defences with telemetry, NOT for source-level orphans. Distinct from feedback_defence_in_depth_budget.
type: feedback
originSessionId: 4654b428-d2fd-4362-9615-f73c1c7648d1
---
**Rule:** When a refactor leaves an enum value, getter, field, abstract class, deprecated impl, "future-use" placeholder, or any other source-level symbol with **ZERO consumer at the moment of the PR**, delete it in the SAME PR. Do not add "kept for forensic optionality" comments. Do not add "/schedule cleanup in 90 days" agent tasks. Do not write "90-day sunset clock" entries in CLAUDE.md / risk tables / changelogs. Become lighter in the same commit that introduced the lightening.

**Why:** 2026-04-25 — during the CalcTape auto-switch rollback (v3.9.0), I (Claude) folded an audit's NICE-TO-HAVE recommendation to keep `OverflowLevel.critical` + the two-tier `OverflowThreshold` strategy + `previousOverflowLevel` field "for forensic optionality, 90-day sunset clock 2026-07-25". User pushback was immediate and correct: "If there's no consumer, what's the point of keeping it? Why are we waiting three months? Things which need to be elevated today should be elevated and be cleared today so that we become lighter over time." The follow-up commit deleted ~150 LOC of dead two-tier plumbing (enum, abstract strategy, deprecated impl, change-detection field) — work that the sunset clock would have deferred for three months while accumulating comprehension debt + sentinel-grep noise + "what is this for?" reader friction.

**How to apply:**
- During a refactor, before writing "kept for forensic optionality" or "X-day sunset clock" in any artifact (code comment, CLAUDE.md, plan, STATUS.md, risk table) — STOP. Ask: does this symbol have a consumer in `lib/` (excluding `_test.dart`/`.g.dart`/`.freezed.dart`/`CLAUDE.md`)? If no — delete it now, in the same PR.
- "If we ever resurrect feature X, we'd want this" is YAGNI — git history has it. Resurrection-day rewrite cost is small; carrying-cost over months is large (sentinel-grep blocklists, reader cognition, build_runner regeneration cycles).
- A `@Deprecated('use Y instead')` annotation with **zero callers** is a delete candidate, not a deprecation. Deprecation is for symbols with current callers needing a migration window. No callers = just delete.
- Audit recommendations like "keep on N-day sunset clock" are NICE-TO-HAVE → push back if they require carrying dead code. Honor `feedback_dont_let_audits_override_user_commitments` — but apply it to the user's lightening intent too.
- This rule is DISTINCT from `feedback_defence_in_depth_budget`. That rule is about PRODUCTION DEFENCE LAYERS (MATPU/billing ACL chains) where you measure catch-rate over 30 days before removal — because removal could let a real failure through. THIS rule is about SOURCE-LEVEL ORPHANS where there's no production traffic to measure. The difference is observability: defence layers have telemetry; orphan enum values do not.
- Decision tree: Is this code reachable from the current call graph? (a) Yes → keep. (b) No, but it's a defence layer with telemetry → 30d sunset per `defence_in_depth_budget`. (c) No, and there's no telemetry to measure → delete in this PR.
