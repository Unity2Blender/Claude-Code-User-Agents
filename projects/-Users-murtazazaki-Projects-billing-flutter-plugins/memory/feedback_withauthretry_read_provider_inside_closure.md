---
name: Read postgrestProvider INSIDE the withAuthRetry closure, never capture outside
description: Capturing `_container.read(postgrestProvider)` before `withAuthRetry` means Strategy 2 client recreation retries against a stale captured PostgrestClient.
type: feedback
originSessionId: 2f07a826-81cd-48ae-ac41-c6677a99d6e5
---
`withAuthRetry` (in `lib/core/services/postgrest_retry.dart`) has a two-strategy recovery path: Strategy 1 = in-place `setAuth()`, Strategy 2 = full `forceRecreateClient()`. Strategy 2 disposes the old `PostgrestClient` and creates a new one — any captured reference to the old client is now stale and will fail when retried.

**How to apply:** Always re-read `postgrestProvider` (or `supabaseRestProvider`) INSIDE the closure passed to `withAuthRetry`:

✅ Correct — re-reads on every retry attempt:
```dart
await withAuthRetry(
  () => _container!.read(postgrestProvider).rpc(
    'register_fcm_token',
    params: {...},
  ),
  timeout: const Duration(seconds: 8),
);
```

❌ Wrong — captures stale client, Strategy 2 retries against disposed reference:
```dart
final postgrest = _container!.read(postgrestProvider);  // captured once
await withAuthRetry(
  () => postgrest.rpc('register_fcm_token', params: {...}),
);
```

Same rule applies to `_container.read(supabaseRestProvider)` and any other provider that wraps a refresh-able client.

Cross-reference: `lib/core/services/postgrest_retry.dart` (`withAuthRetry`, Strategy 2 = `forceRecreateClient()`), `lib/core/services/supabase_rest_client.dart:589-637` (`forceRecreateClient()` body — disposes old PostgrestClient).
