---
name: GOTCHA-038 late final in Riverpod notifier build()
description: Never use late final for fields initialized in Riverpod notifier build() — crashes on keepAlive rebuild or dependency change
type: feedback
---

Never use `late final` for fields initialized in a Riverpod notifier's `build()` method. Use `late` (non-final) instead.

**Why:** Riverpod reuses the same notifier instance on `ref.invalidate()` or dependency-triggered rebuilds (via `ref.watch()` in build). It calls `build()` again on the same Dart object. `late final` prevents reassignment → `LateInitializationError`. This applies to ALL notifiers, but only manifests at runtime for `keepAlive: true` notifiers (autoDispose notifiers create new instances).

**How to apply:** When writing or reviewing any `@riverpod` notifier class, check for `late final` fields that are assigned inside `build()`. Change them to `late` (drop `final`). The fields are private and only assigned in `build()`, so removing `final` has no practical downside. Hidden trigger: `ref.watch(someProvider)` inside `build()` creates a reactive dependency — if that provider changes, `build()` re-runs on the same instance.

**Detection:** grep for `late\s+final` inside `*_notifier.dart` files with `@Riverpod(keepAlive: true)`.

**Discovered:** 2026-03-22 — PartySearchNotifier crash during deferred onboarding flow. Crashed when `DeferredInvoiceEntryPage` called `ref.invalidate(partySearchProvider(...))` after FirmWizard completed.
