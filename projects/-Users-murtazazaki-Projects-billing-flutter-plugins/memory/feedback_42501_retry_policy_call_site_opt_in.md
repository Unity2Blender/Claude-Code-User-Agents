---
name: 42501 retry policy belongs at the call site, not in the global classifier
description: `withAuthRetry`'s `_isJwtExpired` should not blanket-retry all `42501 + permission denied for function`. Use an explicit `retryablePermissionDeniedFunctions` parameter so each caller declares which RPC's 42501s are recoverable.
type: feedback
originSessionId: 2f07a826-81cd-48ae-ac41-c6677a99d6e5
---
`lib/core/services/postgrest_retry.dart::_isJwtExpired` historically returned `true` for any `42501 + permission denied for function` error, which incorrectly retries admin/service-role-only RPCs whose 42501 is the contract working as intended.

**Rule:** The 42501 retry decision must come from the caller, not the global classifier. Add a parameter:

```dart
Future<T> withAuthRetry<T>(
  Future<T> Function() request, {
  Duration timeout = const Duration(seconds: 10),
  Set<String> retryablePermissionDeniedFunctions = const {},
  ...
})
```

Inside the 42501 branch:

```dart
if (e.code == '42501' && e.message.contains('permission denied for function')) {
  if (retryablePermissionDeniedFunctions.isEmpty) return false;
  final fn = _parseFunctionNameFromPermissionDeniedMessage(e.message);
  if (fn == null) return false;            // fail closed on parse failure
  return retryablePermissionDeniedFunctions.contains(fn);
}
```

The function-name parser may live in `postgrest_retry.dart` (it's extraction logic), but the **policy** (which RPCs are retry-safe) must be declared at every call site:

```dart
await withAuthRetry(
  () => postgrest.rpc('register_fcm_token', params: {...}),
  retryablePermissionDeniedFunctions: const {'register_fcm_token'},
);
```

**Why:** A future admin RPC added to the same router (e.g., `cleanup_stale_fcm_tokens` called from a service path) would inherit blanket retry under the old design, masking real permission errors. Call-site opt-in keeps policy local to where the contract is known.

**Migration concern:** Removing the global blanket retry is a behavior change. Audit all existing `withAuthRetry` callers and pass the new parameter explicitly for any that depended on the old behavior. PGRST303, JWT-expired, and RLS-iat-freshness branches are unchanged.

Cross-reference: `lib/core/services/postgrest_retry.dart:380-392` (the over-broad branch), `feedback_evidence_gates_apply_recursively.md` (don't remove safety nets without evidence — audit before deleting blanket retry).
