---
name: GOTCHA-031 Native Method Channel Blocks Dart Future
description: InAppUpdate.startFlexibleUpdate() blocks Dart future until download completes (10-60s), not just consent. Never await in PopInterceptor onIntercept.
type: project
---

## GOTCHA-031: Native Method Channel Blocks Dart Future Until Download Completes

**Pattern**: `await InAppUpdate.startFlexibleUpdate()` in PopInterceptor's `onIntercept` blocks the page pop for the entire download (10-60 seconds), not just the consent dialog.

**Why**: Reading `InAppUpdatePlugin.kt:218-242` reveals the exact native flow:

```
startFlexibleUpdate() native flow:
  1. Shows Play consent dialog immediately
  2. User DECLINES → method channel resolves with error → Dart future completes quickly
  3. User ACCEPTS → download starts → method channel stays PENDING
  4. Download progresses → installUpdateListener fires: pending → downloading → ...
  5. Download COMPLETES → method channel resolves with success → Dart future completes
  6. Download FAILS/ERROR → method channel resolves with error
```

**Key insight**: The Dart `await InAppUpdate.startFlexibleUpdate()` blocks until **download completes** (not consent). For accept, this is 10-60 seconds. For decline, it completes immediately with a `PlatformException(code: 'USER_DENIED_UPDATE')`.

**Consequence**: PopInterceptor awaits `onIntercept` before executing navigation. If you `await startFlexibleUpdate()`, the page freezes for the entire download duration. The user presses back and... nothing visible happens for 10-60 seconds.

**Fix**: Fire-and-forget via `unawaited(startFlexibleUpdate())` + `PopInterceptNavigation.pop`. The page pops immediately, and the Play consent dialog appears on the underlying page (CalcHome). State transitions are driven reactively by `installUpdateListener` stream, not by the return value.

**How to apply**: Any native method channel call that blocks until a long-running operation completes (not just UI interaction) must use `unawaited()` when called from `onIntercept`. Track progress reactively via streams/listeners, not by awaiting the future.

**Related patterns**:
- `_ensureInstallListener()` — lazy stream subscription for `InstallStatus` events
- `_updateInstallTriggered` guard — prevents double `completeFlexibleUpdate()` calls during CalcHome rebuilds
- `ref.listen` (not `ref.watch`) — fires on transitions only, not affected by unrelated provider rebuilds

**Files**:
- `lib/state/providers/update/update_ux_provider.dart` — restructured `startFlexibleUpdate()` + install listener
- `apps/gst_calculator/lib/views/gst_result_page.dart` — `unawaited()` in PopInterceptor Condition 1
- `apps/gst_calculator/lib/views/calc_home.dart` — smart restart via `ref.listen`
