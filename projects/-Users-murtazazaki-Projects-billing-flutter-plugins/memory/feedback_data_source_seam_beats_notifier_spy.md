---
name: Test Riverpod feature behavior via the data-source seam, not by spying on the notifier
description: When a widget triggers a Riverpod notifier action (hydrate, refresh, mutate), assert the downstream effect via the dependency seam (the data source the notifier depends on), not by overriding the notifier with a spy.
type: feedback
originSessionId: 5fbffcbe-4672-4cdb-a9ca-c76034c28dfa
---
For a widget tap → notifier method → data source call chain, prefer overriding the **data source** with a recording fake and asserting on its received calls. Do NOT override the notifier itself with a spy that records its own method invocations.

**Anti-pattern:** Override `reelsRecommendationQueueProvider` with a spy notifier that records `hydrateCalls.add(seed)`. Brittle because:
- Requires re-implementing the notifier's internals to keep observable state.
- Defeats route-local `ProviderScope` override patterns (the spy lives at test root, not in the route scope).
- Refactoring the notifier (renaming methods, splitting hydrate into stages) breaks the test even when behavior is unchanged.

**Pattern:**
```dart
class RecordingTutorialDataSource extends MockTutorialDataSource {
  final List<({int? currentId, String? category, int buildNumber})> windowCalls = [];
  @override
  Future<RecommendationWindow> getRecommendationWindow({
    int? currentId, String? category, required int buildNumber,
  }) async {
    windowCalls.add((currentId: currentId, category: category, buildNumber: buildNumber));
    return _nextWindow;
  }
}
// Assert:
expect(dataSource.windowCalls.last.currentId, isNull);
expect(dataSource.windowCalls.last.buildNumber, 2100);
```

This survives notifier refactors, plays nicely with route-local `ProviderScope` overrides (data source overridden at test root, queue notifier free to be route-overridden), and asserts the actual contract (Worker RPC inputs).

**Why:** Round-4 audit on `videos-tab-majestic-harbor` rejected a planned spy-the-notifier test for Discover's seedless hydrate. The data-source seam (`tutorialDataSourceProvider` fake → `getRecommendationWindow(currentId: null)`) is the right behavioral assertion.

**How to apply:** For any Riverpod feature test where a widget action ultimately fires an external dependency (HTTP, RPC, DB, file IO), test through the boundary between notifier and dependency — not between widget and notifier.
