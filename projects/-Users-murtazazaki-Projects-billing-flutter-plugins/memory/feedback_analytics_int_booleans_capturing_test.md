---
name: GA4 analytics doctrine — int booleans, Map<String, Object> only, async unawaited+catchError, CapturingAnalyticsService for tests
description: Firebase Analytics in apps/gst_calculator accepts only String/int/double param values; encode booleans as 1/0 ints. logEvent signature is Future<void> logEvent(String, Map<String, Object>). Wrap async calls in unawaited(...catchError). Tests use CapturingAnalyticsService, not NoOp, when asserting events. Never log raw user query text.
type: feedback
originSessionId: 5fbffcbe-4672-4cdb-a9ca-c76034c28dfa
---
GA4 / Firebase Analytics rules in `apps/gst_calculator`:

1. **Param value type:** `Map<String, Object>` (NOT `Map<String, Object?>`). The `AnalyticsService.logEvent` signature at `apps/gst_calculator/lib/services/analytics_service.dart:13-16` is `Future<void> logEvent(String name, Map<String, Object> params)`. Firebase Analytics only accepts `String`, `int`, `double` parameter values — `bool` is rejected.
2. **Boolean encoding:** Pass `value ? 1 : 0` (int), never the Dart bool. e.g. `'has_query': hadQuery ? 1 : 0`.
3. **Async safety:** `logEvent` returns `Future<void>`. A synchronous `try { ref.read(provider).logEvent(...) } catch` does NOT catch failures inside the unawaited Future. Use:
    ```dart
    unawaited(
      ref.read(analyticsServiceProvider).logEvent(name, params).catchError((_) {/* decorative */}),
    );
    ```
4. **Tests:** Implement `CapturingAnalyticsService implements AnalyticsService` whose `logEvent` matches the exact signature and records `events.add((name: name, params: params))`. Do NOT use `NoOpAnalyticsService` when asserting events. Assert `params['has_query'] == 1` (int), not `true` (bool).
5. **Privacy:** `debug_log_collector.dart` persists debugPrints to disk. Never log raw user-entered query text. Log `query_length` (int) only.

**Why:** Round-5 audit on `videos-tab-majestic-harbor` plan (2026-05-03) caught the wrong analytics signature, the swallowed-async-error bug, and the privacy leak via debug logs all in one PR. Three distinct gotchas converging on the analytics call site.

**How to apply:** Every analytics call in `apps/gst_calculator` (and code running in this app's context). Test patterns travel with `analyticsServiceProvider` overrides — always favor capturing over NoOp when the test asserts a specific event was emitted.
