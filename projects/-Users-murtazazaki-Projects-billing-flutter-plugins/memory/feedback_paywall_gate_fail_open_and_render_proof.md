---
name: Paywall gate must fail-open + only mark "shown" after wizard actually rendered
description: Discipline for one-time paywall/upsell gates around foreign UI flows. Coordinator preflight short-circuits and exceptions must not block the underlying user action, and must not falsely mark "shown" when the wizard was never displayed.
type: feedback
originSessionId: 1cab8526-61e0-4cc7-81dd-a06e3f52e4fc
---
When wrapping a feature surface (importer wizard, invoice wizard, etc.) with a one-time paywall/teaser gate, two failure modes recur:

1. **Coordinator preflight short-circuits silently** — `PaywallCoordinator.showPaywall` returns null when `userIdProvider` is null (anonymous corner) or when the user is already paid. If the gate naively marks `postAuthTeaserProvider.markShown()` post-await, it persists "user has been shown the paywall" for users who never saw it. Next session they're never re-prompted.
2. **Coordinator/markShown/analytics exceptions block the user action** — if the gate awaits `coordinator.showPaywall(...)` without try/catch, any thrown error skips the underlying intent (importer never opens). The user observes a tap that does nothing.

**Rule.** Every "show this once" paywall/upsell gate must:
- Precompute `hadUserId = ref.read(userIdProvider) != null && it.isNotEmpty` BEFORE the launch.
- Wrap `coordinator.showPaywall(...)` + `markShown()` + analytics in `try { ... } catch (e) { debugPrint('[PAYWALL_GATE] error: $e'); }`.
- Mark `markShown()` ONLY when the wizard route actually rendered AND was dismissed by the user. Coordinator preflight short-circuits (null UID, paid bypass) emit `result: 'bypassed'` analytics with explicit `via:` discriminator; they do NOT mark shown.
- Always return the original `BillingContext` so the underlying action resumes regardless of paywall outcome.

**Why:** 2026-05 audit on `_maybeShowFirstBillingPaywall` flagged that the original `_maybeShowPostAuthTeaser` would have falsely marked SharedPrefs for null-UID users, and any analytics/persistence exception would have blocked the importer launch. Both are silent UX regressions that don't surface in widget tests unless explicitly covered.

**How to apply:**
- New paywall/teaser gates: copy this fail-open + render-proof pattern from `billing_action_gate.dart::_maybeShowFirstBillingPaywall`.
- Existing gates: audit for unconditional `markShown()` post-await. Add the precompute + try/catch + render-proof guard.
- Analytics schema: keep a stable `result` field for dashboard continuity; add `via:` discriminator with `paid_bypass | shown_bypass | session_bypass | no_uid | launch_error | wizard_dismissed | wizard_purchased | wizard_null`.
- Tests: must cover (a) coordinator preflight short-circuit (null UID) → no markShown; (b) thrown exception → no markShown + billingContext returned; (c) wizard rendered + dismissed → markShown happens; (d) branch precedence (session → paid → SharedPrefs).
