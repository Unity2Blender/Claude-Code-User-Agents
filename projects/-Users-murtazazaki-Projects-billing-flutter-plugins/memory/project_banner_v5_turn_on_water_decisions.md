---
name: Banner v5 "turn on the water" — 4 locked decisions + 12 directives
description: Pre-launch Banner v5 rollout contract locked 2026-04-14 after user-led parallel audit (5 skills). Applies to any future change that touches banner.ts caching, killswitch, purge, or DynamicFooter gating.
type: project
status: contextual-history
originSessionId: 28a13cc0-721b-46ea-8bac-ea6f4a198333
---

**Kernel status:** Historical/contextual Banner v5 rollout record. Preserve cache/purge facts when debugging this subsystem, but do not copy its UI flag posture into future UI plans. Current active doctrine is `MEMORY.md`: no UI runtime flags unless the user explicitly asks.

User ran parallel audits with `flutter-architecture-principles`, `design-postgres-tables`, `brainstorming`, `devops-architect`, `test-integration-principles` on 2026-04-14 and surfaced 9 issues on my v1 plan (lucky-launching-lollipop.md). v2 plan locks 4 decisions and 12 directives. The decisions are architectural invariants; future changes must not regress them.

**Why:** my v1 plan missed 3 critical items — cache poisoning in `banner.ts:195-223` (excludeIds forwarded to RPC but NOT in cache key), `placement_config.{categories, intents, enabled}` loaded but never forwarded to RPC, and `admin-purge` guidance misleads operators (doesn't touch `placement-config:*` KV keys or R2 illustration edge cache). User's audit caught all three.

**How to apply:**

**Locked Decisions (must not be violated without re-lock):**
1. **GST rollback / global kill restores legacy footer on next cold start.** If `banner_global_enabled=false`, Worker returns `[]`, `bannerRecommendationProvider` yields `AsyncData(null)`, `_buildBannerOrFooter` in `gst_result_page.dart:340-404` falls back to `DynamicFooter`. Do NOT re-introduce `_GstV2BannerSlot` or any `gst_v2_enabled` hard-branch in `dynamic_footer.dart`.
2. **Pre-launch SOT = `production + flags`.** No test-env gating pre-Play-Store. Post-launch the test-env carve-out activates per `project_tutorials_worker_testenv_carveout.md`.
3. **Flag propagation is cold-start-only** (bounded by `/v1/tutorials/config` `Cache-Control: max-age=300`). Do NOT add mid-session invalidation hooks.
4. **`exclude_ids` uses base-pool caching + per-request filter.** Worker MUST NOT persist exclusion-filtered banner arrays in shared KV. `banner.ts` calls RPC with `p_limit: requestedLimit * 5` and NO `p_exclude_ids`; caches the base pool; post-cache applies `filterAndSlice(pool, excludeIds, limit)` before responding.

**Directive mnemonics (full list in plan file):**
- D1: `gst_result_page.dart` is single owner of GST banner-vs-footer presence
- D4: cache base pool, filter per request (locks decision 4)
- D5: either forward `p_categories`/`p_intents`/honor `enabled=false` in banner.ts, or cut those fields from `placementConfigService.ts` + `bannerCacheKeyV3`. Decision deferred to SQL inspection of `get_banner_recommendation` signature.
- D6: add `make admin-purge-banner-all` composite target (purges `banner:v3:*` AND `placement-config:*`); do not rely on `PATTERN=banner:v3:*` alone
- D7: `admin-purge` does NOT invalidate R2 illustration edge/browser cache; illustration changes require key-suffix bump (A2D-007)
- D8: 3s Promise.race timeout on BOTH `supabase.rpc` AND illustration `fetch` in banner-illustration.ts
- D9: no banner host mounts with `currentBuildNumber: 0`; all 4 hosts adopt `gst_result_page.dart:353-361` AsyncData gate pattern
- D10: canonical pre-launch doc language is "Production Worker + hard-off flags + release-build verification"
- D12: paused Patrol scaffold at `integration_test/helpers/robots/banner_robot.dart` does NOT count as integration coverage; explicitly document absence in `apps/gst_calculator/test/README.md`

**New GOTCHA to register post-implementation:**
- GOTCHA-BANNER-001 — `bannerKillswitchProvider` cold-start-only; mid-session KV flips invisible until restart; by design per locked decision 3.

**Rollback SLA contract:** `<60s Worker-side + next-cold-start user-visible, ≤5 min bound`. Do NOT restate as "<60s" without the cold-start qualifier.

**Cache contract test locked at:** `microservices/tutorials/tests/contract/banner.cache.exclude.test.ts` (5 tests). Regressions break this file first.
