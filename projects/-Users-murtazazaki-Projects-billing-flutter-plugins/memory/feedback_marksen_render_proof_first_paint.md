---
name: paywallSessionTracker.markSeen() must fire at Plans-step first paint, not at coordinator preflight
description: Marking seen at coordinator preflight (before bridge load + wizard push) silently consumes the session cap when the bridge throws. Move to Plans step's post-frame callback for true render-proof.
type: feedback
originSessionId: e92ff6ad-9951-46b2-b2c0-38710165b402
---
`paywallSessionTrackerProvider.markSeen()` at `apps/gst_calculator/lib/services/paywall_coordinator.dart:227` fires BEFORE `loadIndianBridgeFn(ref)` at line 235, BEFORE `mounted` re-check at 236, BEFORE `showWizardFn` at 238. If the Indian bridge throws (timeout, non-StateError), context unmounts, or push fails, the user sees nothing yet `paywallSessionTracker == true` for the rest of the session — silent suppression of subsequent paywall attempts.

**Why:** Violates `feedback_paywall_gate_fail_open_and_render_proof.md` ("mark the SharedPrefs 'shown' flag ONLY when the wizard actually rendered"). The user explicitly called this out as a P0 hazard during KPI Checkpoint planning — a fail-CLOSED hazard (denies a user a future, real paywall in the same session) on top of the new fail-open contract for KPI checkpoint guard.

**How to apply:** Move `markSeen()` to `PlansStep`'s post-frame callback in `initState` (`lib/billbook/paywall/pages/paywall_wizard/views/ui/_shared/steps/plans_step.dart`). PlansStep painted == paywall actually rendered. Existing 9 trigger callers handle null returns; behavior is strictly safer (no silent caps).

**Cross-boundary mechanism:** PlansStep is in root `lib/`; `paywallSessionTrackerProvider` is in app `apps/gst_calculator/lib/state/`. Root cannot import app, so use a callback seam: add `onPaywallPlansStepRendered: void Function()?` to `BillingLibraryCallbacks` (the existing root↔app callback singleton). PlansStep fires the callback in post-frame; app wires it to `paywallSessionTrackerProvider.notifier.markSeen()` in main.dart.

**Defense-in-depth:** Any code that uses the tracker (e.g., KPI checkpoint guard) should additionally check tracker before/after exception during paywall fire — emit `fire_failed_after_marked_seen` alarm if tracker flipped despite exception. Surfaces regression if anyone re-introduces the early-markSeen anti-pattern.

**Note:** `PlansStep` currently extends `ConsumerWidget` (no `initState`). Conversion to `ConsumerStatefulWidget` is the prerequisite Step 0a for the render-proof fix.
