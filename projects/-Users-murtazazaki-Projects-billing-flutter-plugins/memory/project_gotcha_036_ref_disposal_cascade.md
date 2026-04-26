---
name: GOTCHA-036 Ref Disposal During Purchase Flow
description: Reactive watch cascade disposes paywallCoordinator ref mid-flight when billingContext resolves asynchronously (was-premium accounts)
type: project
---

**GOTCHA-036 (Critical):** `paywallCoordinatorProvider.build()` watched `paywallBridgeProvider`, which watched `razorpayCheckoutPrefillProvider`, which watched `billingContextProvider`. When billingContext resolved async (was-premium accounts with Supabase records), the cascade rebuilt everything, disposing the coordinator's ref while `buildNotes` closure held a stale reference → `StateError`.

**Fix (4 layers):**
1. Removed `ref.watch(paywallBridgeProvider)` from coordinator `build()` — reads on-demand instead
2. Removed `ref.watch(prefill/config)` from bridge `build()` — `getOrCreate()` handles staleness
3. Eagerly resolve `uid`/`billingId` in `_loadIndianBridge()` before creating closure (GOTCHA-033)
4. Added `ref.mounted` guards after every async gap in `_showPaywallInternal` (GOTCHA-034)

**Why "previously premium" only:** These accounts have real Supabase records, so `billingContextProvider` resolves asynchronously. Fresh accounts resolve instantly/null → no cascade.

**How to apply:** When adding watches to keepAlive service notifiers, verify no downstream cascade can dispose the ref during active async operations. Prefer on-demand reads over watches for service-pattern notifiers.
