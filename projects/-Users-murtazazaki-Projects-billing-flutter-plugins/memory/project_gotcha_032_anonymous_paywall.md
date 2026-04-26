---
name: GOTCHA-032 Anonymous Paywall Auth Guard
description: Anonymous Firebase users could reach Razorpay payment — fixed with 3-layer defense (anonymous check + billing context guard + notes builder StateError)
type: project
---

GOTCHA-032: Anonymous users with `isFullyAuthenticated=true` could reach Razorpay checkout.

**Root cause chain:**
1. `startPurchase()` only checked `isFullyAuthenticated` (= `isSignedIn && isClaimValidated`), not `isAnonymous`
2. No billing context guard between auth gate and `onPurchase` call
3. `RazorpayNotesBuilder` silently accepted empty UIDs (`?? ''`)
4. `PaywallCoordinator._runAuthTransition()` uses `skipBillingSetup: true`

**Fix (3-layer defense-in-depth):**
1. Explicit `authState.isAnonymous` check before existing auth gate in `startPurchase()`
2. `billingContextProvider` guard with 15s timeout and `preparingPayment` phase (revived dead enum)
3. `RazorpayNotesBuilder.build()` throws `StateError` on null/empty uid or billingId

**Why:** Financial impact — payment collected but entitlement not granted (webhook reconciliation fails with empty UIDs).

**How to apply:** Any new payment flow entry point must check both `isFullyAuthenticated` AND `!isAnonymous` AND billing context availability before invoking payment SDK.
