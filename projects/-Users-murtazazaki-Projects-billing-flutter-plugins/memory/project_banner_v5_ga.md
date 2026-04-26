---
name: Banner v5 GA milestones (CP-01..19)
description: Post-GA snapshot of what shipped per checkpoint + where to grep to prove each piece is present
type: project
status: contextual-history
originSessionId: 519b0f7a-5681-477f-81a4-0ccaf3994e4a
---

**Kernel status:** Historical/contextual Banner v5 GA record. Use for debugging shipped banner checkpoints only. Its rollout flag sequence is superseded as general UI doctrine by `MEMORY.md`: no UI runtime flags unless the user explicitly asks.

Banner v5 (Channels 3 + 5) shipped in 19 checkpoints on 2026-04-13 on branch `banner-impl`. Plan file: `.claude/plans/cozy-munching-thacker.md`. Parent: `docs/BANNERS_V5_REELS.md` (105 directives).

**Why:** Unified tokenized banner recommender for GST hero + Bills/Items/Parties tabs; retires the legacy DynamicFooter 3-state funnel behind a KV killswitch.

**How to apply:** When asked about banner v5 wiring or debugging a specific CP, cite the resumption hooks below. Each CP has a grep that proves it shipped.

### CP resumption hooks
- CP-01 intent seed: `git grep 000253_tutorial_intent_seed`
- CP-02 v3 watch-state RPC: `git grep 000254_get_user_tutorial_views_v3`
- CP-03 get_tutorial_by_id: `git grep 000255_get_tutorial_by_id`; Dart `TutorialDataSource.getTutorialById`
- CP-04 anon Worker sentinel: `microservices/tutorials/tests/contract/anon_tab_banner.contract.test.ts`
- CP-05 rollbacks + preflight: `supabase/rollbacks/rollback_{000246,000248,000249}.sql` + R-11..14 in preflight-production.sql + `make db-rollback-test`
- CP-06 Worker smoke gate: `make smoke-test-worker` + expanded verify-deploy.sh
- CP-07 L55 wiring: `isEngagedForBannerFilterFull` in `banner_recommendation_provider.dart`
- CP-08 tutorials.reel.*: `_launchReelForTutorial` in `feature_navigation_coordinator.dart`; `_adaptTutorialsReel` in `legacy_target_adapter.dart`
- CP-09 reel CTA optimistic: `onCtaTapRegistered` on ReelVideoCard; `_registerReelCtaTap` in `reels_launcher.dart`
- CP-10 GST v2 flag gate: `_GstV2BannerSlot` in `dynamic_footer.dart`; `banner_killswitch_provider.dart`
- CP-11 GST baseline: `docs/banner_v5_gst_baseline.json` (placeholder — fill before prod flip)
- CP-12 StatsGridVariant: `enum StatsGridVariant` in `stats_grid.dart`; `sales_amount_provider.dart`
- CP-13 Bills injection: `_buildBillsBannerSlot` in `dashboard_home_tab_wrapper.dart`
- CP-14 deploy gate: `microservices/tutorials/scripts/phase1-deploy-gate.sh` + `make phase1-gate{,-test,-prod}`
- CP-15 integ tests: `integration_test/banner/banner_test.dart` + `banner_robot.dart` + `make test-integ-banner`
- CP-16 goldens: `test/billbook/tutorials/banner/goldens/banner_card_golden_test.dart` (`@Tags(['advisory'])`)
- CP-17 manifest + analytics: search `test/manifest.json` for `billbook.tutorials.banner`; 5 events in `reel_analytics.dart`
- CP-18 Items injection: `_buildItemsBannerSlot` in `items_tab_wrapper.dart`
- CP-19 Parties Phase D: `parties_aggregate_provider.dart`; `_buildPartiesStatsRow` in `parties_tab_wrapper.dart`

### Deploy gate sequence (post-merge)
1. Capture 30-day GST baseline → fill `docs/banner_v5_gst_baseline.json`
2. `make phase1-gate-test` (dry-run)
3. Flip `gst_v2_enabled` test → 24h observe → prod
4. Flip `banner_tab_bills_enabled` test → Week-1 observe → prod
5. Repeat for `banner_tab_items_enabled` (Phase C) then `banner_tab_parties_enabled` (Phase D GA)

### Open followups (v5.1+)
- Wire the 5 banner analytics events at their fire points (params are ready in `reel_analytics.dart`)
- Generate the 30 golden PNGs (`flutter test --update-goldens test/billbook/tutorials/banner/goldens/`)
- Replace compile-time `defaultBannerFlags` with a Worker `/v1/tutorials/config` fetch
- Finalize Metabase roll-up for CTR + dismiss monitoring
