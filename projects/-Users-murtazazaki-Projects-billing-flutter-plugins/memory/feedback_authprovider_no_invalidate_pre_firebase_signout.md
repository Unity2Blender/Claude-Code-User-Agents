---
name: Don't invalidate authProvider before Firebase signOut
description: AuthNotifier.build() reads FirebaseAuth.instance.currentUser synchronously; pre-Firebase invalidation rebuilds with the same signed-in user.
type: feedback
originSessionId: 3825e5fa-e0a0-4ee5-8165-f00dc0651494
---
On cross-tab logout, sign-out broadcast handlers, or any "force auth reset" path, do NOT call `container.invalidate(authProvider)` before `FirebaseAuth.instance.signOut()` has actually fired.

**Why:** `AuthNotifier.build()` at `lib/state/providers/auth/auth_notifier.dart:91-99` reads `FirebaseAuth.instance.currentUser` synchronously. If you invalidate `authProvider` while Firebase still considers the user signed in (because `signOut()` hasn't run yet, or the broadcast was sent from another tab where Firebase IS signed out but THIS tab still has live Firebase state), the rebuild populates `authProvider` with the same UID/claims it had before invalidation. Net effect: invalidation cycle without state change; auth gate remains "signed in" until next route change.

**The canonical reset path on the receiver tab is the FirebaseAuth stream**, not direct provider invalidation. Firebase's auth state listener drives `AuthNotifier` reset when `signOut()` fires anywhere on the same Firebase project; cross-tab logout works because that signal propagates via the Firebase SDK, NOT because the receiver tab manually invalidates `authProvider`.

**How to apply:**
- Cross-tab logout handlers (`apps/billing-web/lib/app/deferred_init.dart` BroadcastChannel listener, mobile equivalents): invalidate `billingContextProvider` ONLY. Trust Firebase to drive `authProvider` via stream.
- `performSignOut()` at `lib/state/providers/auth/sign_out_action.dart:36-49` — this is the canonical pattern: `ref.invalidate(billingContextProvider)` then `await FirebaseAuth.instance.signOut()`. Do NOT add `ref.invalidate(authProvider)` to it.
- If a future requirement genuinely needs synchronous auth reset (e.g., security-critical session kill), the correct fix is `await FirebaseAuth.instance.signOut()` THEN check `authProvider` rebuilds — not `invalidate` before.

**Source:** 2026-05-06 plan rejection. A Round-3 interview answered "invalidate both billingContext + authProvider on cross-tab logout" — the answer was wrong because it did not account for `AuthNotifier.build()`'s synchronous Firebase read. The user's audit caught it before any code was written.
