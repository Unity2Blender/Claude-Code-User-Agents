---
name: overrideWith signature varies by provider type
description: Functional Provider/FutureProvider/StreamProvider keep `(ref) =>` for overrideWith; class-based generated (Async)NotifierProvider use zero-arg `() =>`. Don't mass-sweep.
type: feedback
originSessionId: 3825e5fa-e0a0-4ee5-8165-f00dc0651494
---
When migrating Riverpod test overrides, do NOT mass-convert all `provider.overrideWith((ref) => …)` call sites to zero-arg `() => …`. The signature depends on the provider type:

- **Functional `Provider/FutureProvider/StreamProvider`** — `overrideWith((ref) => value)` retains the `ref`-arg form. This signature was NOT changed in Riverpod 3.0 → 3.2.1.
- **Class-based generated `(Async)NotifierProvider`** — `overrideWith(() => Notifier())` is the zero-arg factory form. This is what `subscriptionProvider` (an `AsyncNotifierProvider` per `lib/state/providers/subscription/subscription_provider.g.dart:242-248`) uses.
- **Build-only mocking on a class-based notifier** — `overrideWithBuild((ref, self) => …)` exists alongside `overrideWith` (Riverpod 3.0+) when you want to mock `build()` while preserving the real notifier's mutator methods.
- **`overrideWithProvider`** — REMOVED in Riverpod 3.0; migrate to `overrideWithValue` for state-only injection.
- **`family.overrideWith`** — DEPRECATED in 3.2.0 in favor of `family.overrideWith2` (callback now takes the family argument). Will be renamed back to `overrideWith` in 4.0.

**Why:** A 2026-05-06 plan rejection. A Round-1 interview answered "sweep all 25 sites in one pass" based on the assumption that `(ref) => …` was uniformly invalid in 3.2.1. That assumption was wrong — only class-based notifier sites were affected. Mass-converting functional providers would have broken 14+ working call sites.

**How to apply:** When `flutter analyze` reports `overrideWith` signature errors, inspect the underlying provider type FIRST. If it's a generated `*Provider.create() => *Notifier()` class-based notifier (visible in the corresponding `*.g.dart`), migrate that site to zero-arg. If it's a functional `Provider/FutureProvider/StreamProvider`, leave the `(ref) => …` form alone. Do not migrate sites that pass analyzer.

**Source signatures:**
- `pub.dev/documentation/riverpod/latest/riverpod/AsyncNotifierProvider/overrideWith.html` — `Override overrideWith(NotifierT Function() create)`
- Riverpod 3.0 changelog — `Stream/FutureProvider.overrideWithValue was added back`; `family.overrideWith2` deprecated in 3.2.0.
