---
name: Root↔app package boundary is structural — root cannot import from apps
description: lib/ is the shared root package; apps/<app>/ depend on root via path. Root code MUST NOT import from apps. Use BillingLibraryCallbacks or abstract Provider seams as the injection pattern.
type: feedback
originSessionId: e92ff6ad-9951-46b2-b2c0-38710165b402
---
Root `lib/billbook/...` and `lib/state/...` cannot import from `apps/<app>/...`. The pubspec workspace at root declares `workspace: [apps/gst_calculator, apps/billing-web, apps/tutorials_admin]`; each app has `dependencies: billing_flutter_plugins: path: ../../`. Imports in the reverse direction would create a cycle and don't work in pub workspaces.

**Why:** During the KPI Checkpoint plan, I drafted arm sites in shared `lib/billbook/invoicing/...` calling `ref.read(kpiCheckpointProvider.notifier).arm(...)` where the provider was app-local at `apps/gst_calculator/lib/state/...`. The user flagged this as a P0 architecture violation. Same hazard surfaced when I tried to call `paywallSessionTrackerProvider.markSeen()` from the root `plans_step.dart` — root code cannot reach the app-local provider.

**How to apply:** When a feature spans both layers, split it:
1. Pure types + decision logic + reusable widgets → root `lib/billbook/<domain>/`
2. App-specific wiring (Riverpod overrides, Firebase, RevenueCat) → `apps/<app>/lib/state/<domain>/`
3. Inject app-side dependencies into root via either:
   - **Abstract Provider seams** that throw `UnimplementedError` until app overrides (the KPI Checkpoint pattern)
   - **`BillingLibraryCallbacks.instance.<callback>?.call()`** — the existing root↔app callback singleton at `lib/navigation/callbacks/billing_library_callbacks.dart` (used today for `onAnalyticsEvent` because root can't import `firebase_analytics`)

**Sentinel test required:** A `static_import_sentinel_test.dart` at `test/architecture/` greps every `lib/**/*.dart` for both `import 'package:gst_calculator/...'` AND relative `'../apps/...'` patterns. Runs in critical tier. Without this, a developer can accidentally relative-import an app file and bypass package-import linting.
