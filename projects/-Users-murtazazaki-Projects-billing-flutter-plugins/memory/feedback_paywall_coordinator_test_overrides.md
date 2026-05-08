---
name: PaywallCoordinator tests need explicit userId + geo overrides or they flake
description: Required ProviderContainer overrides for any test path that reaches PaywallCoordinator.showPaywall — without them, the coordinator preflight masks behavior and Indian bridge loaders flake the suite.
type: feedback
originSessionId: 1cab8526-61e0-4cc7-81dd-a06e3f52e4fc
---
`PaywallCoordinator.showPaywall` (`apps/gst_calculator/lib/services/paywall_coordinator.dart`) has two preflights that silently change test outcomes when not overridden:

- Line 204-213: returns null + emits SnackBar when `ref.read(userIdProvider)` is null/empty. Tests reading uninitialized providers hit this branch and never reach `showWizardFn`.
- Line 215: returns null when `isPaidSubscriberProvider` is true.
- `geo_provider.dart:37-42`: `isIndianUserProvider` defaults to India (`true`) while geo IP is loading. Tests that don't override this hit `loadIndianBridgeFn` which awaits real billing context (10s timeout in `_loadIndianBridge`).

**Rule.** Any test that touches a code path leading to `PaywallCoordinator.showPaywall` (directly or via `_maybeShowFirstBillingPaywall` / `requireBillingContext`) MUST override the following providers via `ProviderContainer.test(overrides: [...])`:

```dart
ProviderContainer.test(overrides: [
  userIdProvider.overrideWithValue('test-uid'),       // non-null
  userContextProvider.overrideWith(...),               // resolved
  subscriptionProvider.overrideWith(...),              // free or paid as test demands
  postAuthTeaserProvider.overrideWith(...),            // shown / not-shown
  paywallSessionTrackerProvider.overrideWith(...),     // seen / not-seen
  backendReadyProvider.overrideWithValue(Future.value()),
  isIndianUserProvider.overrideWithValue(false),       // OR true with fake loadIndianBridgeFn
]);
```

AND in `setUp`/`tearDown`:
```dart
setUp(() {
  PaywallCoordinator.resetForTesting();
  // optionally: PaywallCoordinator.showWizardFn = mySpy;
  // capture debugPrint here
});
tearDown(() {
  PaywallCoordinator.resetForTesting();
  // restore debugPrint here
});
```

**Why:** 2026-05 audit flagged that without the userId override, migrated tests would never reach `showWizardFn` because the coordinator's null-UID preflight at line 204 returns null first. Without the geo override, the Indian bridge loader runs and tests flake on real billing-context resolution. Both are reset in BOTH lifecycle hooks (not just one) per `FLAKY-STATIC-001`.

**How to apply:** Use this override block as a copy-paste prelude for any new `PaywallCoordinator`-adjacent test. Document the override block in `apps/gst_calculator/test/CLAUDE.md` if it doesn't already exist there. Prefer a coordinator/target-handler-level regression test over a full reel-video UI pump for CTA-chain coverage — faster + less flaky while still proving the chain.
