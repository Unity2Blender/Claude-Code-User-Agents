---
name: Cold-start interleaving is usually a real ref.read reactivity bug, not flakiness
description: Bootstrap providers using ref.read on auth/JWT state miss reactive `null → token` recovery. Don't reach for Clock injection without checking the bootstrap chain reactivity.
type: feedback
originSessionId: 3825e5fa-e0a0-4ee5-8165-f00dc0651494
---
When tests fail with "cold-start interleaving" symptoms (boot chain reads stale `null` JWT, billing context never resolves, retries don't recover), the root cause is usually a `ref.read` in a bootstrap provider that should be `ref.watch` or invalidation-driven — NOT flakiness or time-based non-determinism.

**Why:** `lib/state/providers/auth/jwt_bootstrap_provider.dart:97` reads `ref.read(jwtStateProvider)`. Tests at `test/foundational/cold_start_jwt_arrival_test.dart:1-9` and `:72-85` require reactive `null → token` recovery: when JWT arrives after the bootstrap function has already executed, the bootstrap must rebuild. With `ref.read`, the bootstrap snapshot is frozen; with `ref.watch`, it rebuilds reactively.

`billingContextProvider` at `lib/state/providers/billing_context_provider.dart:161-188` awaits `jwtBootstrapProvider.future`. If the bootstrap is non-reactive, the billing chain remains stuck on the first `null` read.

**How to apply:**
- When a "cold-start" or "boot interleaving" test fails, FIRST grep the bootstrap chain for `ref.read(...)` on auth/JWT/identity providers. Convert to `ref.watch` (full reactive) or `ref.listen + ref.invalidateSelf()` (invalidation-driven) based on the surrounding code shape.
- `ref.watch` is safe for AsyncNotifier readers; the provider rebuilds when the watched state transitions.
- DO NOT add `Clock.fixed()` plumbing as a workaround. Clock injection masks the architectural bug; the test will still flake when the real cold-start path runs because the dependency edge is wrong.
- Once the source fix lands, run targeted cold-start tests to confirm green: `flutter test test/foundational/cold_start_jwt_arrival_test.dart -p vm` and `flutter test test/foundational/cold_start_interleaving_pbt_test.dart -p vm`.

**Source:** 2026-05-06 plan rejection. The original plan proposed `Clock.fixed()` injection per `CHAIN-CIRCUIT-001` exemplar at `test/foundational/billing_context_retry_test.dart`. The user's audit identified the actual fix shape: `ref.read` → reactive in `jwt_bootstrap_provider.dart:97`. Time injection was a workaround, not the cure.
