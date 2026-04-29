---
name: Use SupabaseRestClient.instance for token refresh, never the Riverpod provider
description: `supabaseRestProvider` and `postgrestProvider` are Layer 1.5 transport guards that throw StateError on null JWT. Token-management methods must call the singleton directly to bypass that guard intentionally.
type: feedback
originSessionId: 2f07a826-81cd-48ae-ac41-c6677a99d6e5
---
`postgrestProvider` and `supabaseRestProvider` throw `StateError('MATPU: Supabase blocked — no authenticated JWT')` when `ref.watch(jwtStateProvider)` returns null. See `lib/state/providers/supabase_provider.dart:110-160`. This is the documented Layer 1.5 transport chokepoint enforced by ADR-001.

**Why:** When refreshing a JWT (e.g., after expiry, account switch, or post-validation), the calling code may be running at a moment when `jwtStateProvider` is null or about to be repopulated. Reading the provider would throw mid-recovery and crash the refresh path. The singleton `SupabaseRestClient.instance.refreshAuth()` and `forceRecreateClient()` deliberately bypass the provider guard so token management can run regardless of provider state.

**How to apply:** When writing or editing token-refresh / re-validation code in `auth_notifier.dart`, `jwt_refresh_notifier.dart`, `withAuthRetry`, or any FCM/auth recovery path:

- ✅ `await SupabaseRestClient.instance.refreshAuth();` then `ref.read(jwtStateProvider.notifier).update(SupabaseRestClient.instance.currentJwt);`
- ❌ `await ref.read(supabaseRestProvider).refreshAuth();` — this throws if `jwtStateProvider` is null at the moment of the read

Order matters: refresh first via the singleton, THEN write the post-refresh JWT into `jwtStateProvider` so downstream provider reads see the new token.

The provider path (`ref.read(supabaseRestProvider)`) is correct for normal `.from()` queries and `.rpc()` calls — they SHOULD throw if a request is attempted with no JWT. Only token-management methods (refreshAuth, forceRecreateClient, updateToken) should bypass via the singleton.

Cross-reference: `supabase_provider.dart:142-160` (the throw site), `supabase_rest_client.dart:139-141` ("Token management methods are called via the singleton directly, not through this provider, so they are unaffected by this guard"), `docs/architecture/ADR-001-matpu-transport-guard.md`.
