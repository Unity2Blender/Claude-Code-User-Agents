---
name: Banner water-on-high leak plugging (ADR-007)
description: Parent plan ADR for plugging 4 leaks surfaced by ADR-006 water-on-high pivot; Source→Destination Contract reframe; 15 locked decisions; 3-tier depth-in-defence
type: project
status: contextual-history
originSessionId: 1636fa18-93d5-4e42-b373-51c0d539de25
---

**Kernel status:** Historical/contextual banner water-test record. Keep the source/destination and schema-validation lessons for banner debugging. Current active water-test doctrine is `MEMORY.md` plus `feedback_seibel_water_test_requires_bucketing.md`: ship small plus targeted evidence, not speculative UI flags.

ADR-006 turned `p_include_placeholder DEFAULT TRUE` (000357) — water on high. Release build 3.2.3072+1 surfaced 4 leaks: parse NPE (100% placements), `reels.browser` CTA dead-end, prewarm cohort timing gap, 29× slow latency.

**ADR-007 fix (all in one coupled deploy):**
- **Tier 1 DB** — Migration 000358 CHECK on `tutorials_reels` (NOT `tutorial_banners` — v2 audit correction): placeholder rows need non-null `category` + `cta_config`. `title` redundant (NOT NULL at column level).
- **Tier 2 Worker** — `schema_violation` as global `BannerOutcome` (bannerOutcome.ts + isInfraOrDataFailure + 100% sample). `filterSchemaValidRows` validator between SRP filter and cache. NO COALESCE (dead code — 000357:87 already COALESCEs override).
- **Tier 3 Flutter** — BannerRecommendation.title/category/placementId/ctaConfig widened to nullable. Provider throws `BannerTransportException(reason: decodeError)` — extends existing transport enum (NOT a new sealed class, GOTCHA-V4-003 preserved). Absence notifier gets `decodeError` case. 5 widgets get caller-side null fallbacks.

**Source→Destination Contract (ADR-007 §2 — load-bearing):**
- Sources (many): GST banner, tab banner, FCM push, static fallback, compact helper row, future Reels tab.
- Destination (one): Reels surface (ReelsView + ReelPlaceholderCard + swipe graph).
- `reels.browser` is source-agnostic; source metadata (source_placement, source_kind, category, surface_id) threaded, not baked into target identity.

**ADR-006 trigger #2 amended:** parse NPE / decodeError / schema_violation across placeholder rows is designed water-test surface, does NOT trigger reversal.

**Verify:**
- `/v1/admin/health-data-sentinels` reports `banner_outcomes_count: 19` and `banner_outcomes` list includes `schema_violation`.
- No `[BANNER_REC] fromJson failed` in release build logs.
- GST fallback tap opens ReelsView with `source_kind=static_fallback` analytics.

**Plan:** `.claude/plans/i-flutter-29828-main-start-polished-cat.md` (v3.0).
**Committed follow-up:** ADR-008 pool-cohort RPC (`4 round-trips → 1`).
