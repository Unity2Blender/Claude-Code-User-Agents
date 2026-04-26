---
name: Smart Update UX — billing-aware probability gates
description: Update UX threshold redesign — reduced GST gates (15%/10%), added billing touchpoints (50%), ProGuard fix (GAP-P1)
type: project
---

Smart Update UX shipped in `fe5cd1b1` on `feat/tutorial-engine-v5`.

**Why:** 90% of app updates target billing features, but update prompts only fired at GST calculator touchpoints — causing survivorship bias (GST-only users interrupted, billing power users missed).

**How to apply:** When modifying session gates, update thresholds, or adding new update touchpoints:

- Thresholds in `lib/state/providers/session/session_gate_provider.dart`:
  - `isFlexibleUpdatePromptSession` = 15% (GST Result back)
  - `isImmediateUpdateSession` = 10% (CalcHome exit, floor)
  - `isBillingUpdateSession` = 50% (billing actions)
- Billing update gate utility: `apps/gst_calculator/lib/utils/billing_update_gate.dart`
- `tryBillingUpdatePrompt()` has 2.5s delay + double state re-check after delay
- `tryBillingTabReturnUpdate()` fires immediately on billing→calc tab switch
- Invoice wizard at `invoice_wizard_page.dart:219` uses `isBillingUpdateSession` (migrated from `isFlexibleUpdatePromptSession`)
- ProGuard: Play Core `appupdate`/`install`/`common`/`listener`/`tasks` classes kept (was only `review`)
- 75 tests in `pop_interceptor_session_gating_test.dart` (Group J = billing gates + cross-touchpoint mutual exclusion)

**Future:** Reels `update_app` CTA (user-initiated, 100% immediate) — DB schema exists, Flutter client not yet built. Plan: `.claude/plans/elegant-bubbling-gray.md`
