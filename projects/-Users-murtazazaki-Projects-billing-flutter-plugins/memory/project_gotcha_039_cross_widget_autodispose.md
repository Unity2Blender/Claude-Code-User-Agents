---
name: GOTCHA-039 Cross-Widget Auto-Dispose Provider
description: Auto-dispose Riverpod provider used cross-widget with one-shot ref.read() disposes mid-async — never use firmWizardProvider from invoice editor
type: project
---

## GOTCHA-039: Cross-Widget Auto-Dispose Provider Disposal

**Pattern**: An `@riverpod` (auto-dispose) family provider is used from a DIFFERENT widget tree via `ref.read()` (one-shot, no watcher). The provider has zero listeners → Riverpod schedules disposal → during the async gap, the notifier's `ref` becomes stale → `StateError: Cannot use Ref after disposed`.

**Example**: Invoice Editor's `_openBannerGstSheet()` did `ref.read(firmWizardProvider(false).notifier)` → called `loadExistingFirm()` (async) → provider disposed mid-flight.

**Why:** `ref.read()` doesn't create a persistent subscription. Unlike `ref.watch()`, it doesn't keep the provider alive. Riverpod sees zero listeners and disposes the provider instance between frames.

**How to apply:**
- NEVER use `ref.read()` on an auto-dispose provider from a different widget tree
- For cross-widget data loading, use direct postgrest queries or a dedicated FutureProvider
- The fix pattern: load data via `postgrestProvider.from('table').select()`, build local state, manage via `StatefulBuilder`
- Correct usage: `ref.watch()` keeps provider alive (only from within the owning widget tree)

**Fixed in:** Workstream 8, Fix B1 (commit after ab1ad2e8)
