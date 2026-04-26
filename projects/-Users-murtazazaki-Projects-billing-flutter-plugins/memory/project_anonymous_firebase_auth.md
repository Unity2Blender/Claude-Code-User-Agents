---
name: Anonymous Firebase Auth for GST Calculator
description: Fresh users get signInAnonymously() at startup for FCM/Analytics/RevenueCat identity. MATPU guards block Supabase. AuthNotifier skips validation for anonymous users. Account linking via linkWithCredential() preserves UID.
type: project
---

Fresh GST Calculator users now get `signInAnonymously()` in `main()` before `runApp()`. Provides Firebase UID from frame 1 for FCM push notifications, Firebase Analytics, Crashlytics identity, and RevenueCat tracking.

**Why:** Firebase console showed anonymous users only from v2.0 (legacy codebase). Current codebase never called `signInAnonymously()`. 50%+ of users who never sign in were completely invisible — no FCM, no analytics, no conversion tracking.

**How to apply:**
- AuthNotifier.build() short-circuits for `user.isAnonymous` — returns `isClaimValidated: false` WITHOUT scheduling `validateCustomClaims()`
- `_onAuthStateChanged()` new-user branch guards `if (!user.isAnonymous)` before scheduling validation
- Account linking: `anonymousChanged && !user.isAnonymous` triggers validation after `linkWithCredential()` upgrade
- DebugAuth guard: `currentUser != null && !currentUser.isAnonymous` (doesn't skip anonymous users in debug mode)
- FCM stays gated behind onboarding completion (2nd session) — intentional UX decision to avoid notification permission dialog on first impression
- Cloud Function `handleUserCreate` keeps skip behavior for anonymous users — zero server-side footprint
- `kAccountLinkingSteps` constant for AuthTransitionSheet when upgrading anonymous → Google
- Architecture was already prepared: `AuthService.signInWithGoogle()` has `linkWithCredential()` + fallback, `SignInPage` captures `_capturedAnonymousUid`, `AuthTransitionNotifier` checks `authState.isAnonymous`
- 5 critical sentinel tests in `test/features/auth/anonymous_validation_bypass_test.dart`
- 179 auth + MATPU tests pass with zero regressions
