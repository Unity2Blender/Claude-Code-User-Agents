---
name: Auth Pipeline Reactive JWT Fix
description: jwtStateProvider bridges singleton JWT into Riverpod — fixes postgrestProvider cache deadlock after anonymous→Google sign-in
type: project
---

**Fix shipped 2026-04-06** on `feat/tutorial-engine-v5` (commit `5063b1d4`).

Two production bugs fixed by making `postgrestProvider` reactive:
1. "Preparing workspace..." timeout — provider chain deadlocked because postgrestProvider cached MATPU StateError from anonymous state
2. "We are still verifying your account" stuck 24h — `validateCustomClaims()` backup RLS query hit same poisoned provider

**Key change:** `jwtStateProvider` (keepAlive Notifier, starts null) — `JwtRefreshNotifier` writes to it after every `updateToken()`/`refreshAuth()`. Both `postgrestProvider` and `supabaseRestProvider` now `ref.watch(jwtStateProvider)` instead of reading `SupabaseRestClient.instance.currentJwt` directly.

**Why:** Singleton reads are invisible to Riverpod's dependency graph. No `ref.watch()` = no invalidation on JWT change = cached error persists forever.

**How to apply:** Any future provider that needs to react to auth state changes should watch `jwtStateProvider` (or a provider that watches it), never read `SupabaseRestClient` singleton directly.

**Also changed:**
- Removed `linkWithCredential` entirely — always `signInWithCredential` for anonymous→Google (user decision: anonymous data doesn't matter)
- Phase-aware `retry()` in AuthTransitionNotifier — uses `failedAtPhase` to skip completed phases
- NeedsClaims gate now has "Retry" SnackBarAction (was passive-only snackbar)
- Implemented kAccountLinkingSteps for anonymous user auth transition labels
