---
name: Banner System v2 Overhaul (2026-04-17)
description: 85 locked decisions from 10-round interview + 6-agent audit for banner bug-fix + schema foundation build-out
type: project
status: contextual-history
originSessionId: ff28ab3e-695e-4a2b-9395-c64f699f9424
---

**Kernel status:** Historical/contextual Banner v2 plan record. Do not treat its pre-launch flag posture as current UI doctrine. Current active doctrine is `MEMORY.md`: no UI runtime flags unless the user explicitly asks.

**Plan file:** `/Users/murtazazaki/.claude/plans/banner-system-v2-sparkling-hinton.md`
**Audit agents:** devops-architect, flutter-architecture-principles, design-postgres-tables, brainstorming, ui-styling, test-integration-principles (6 agents, 146 new findings)

**Why:** User directive "Schema design, design rules for table scale, and foundation laying is very much important at this point" elevated scope from bug-fix (3 migrations) to foundation build-out (10 migrations: 000267-000276) covering banner_type ENUM, status FSM, cta_config_override, variant_key, placement_category_affinity M2M, banner_impression_count, referrer_banner_id, retired_at, banner_audit_log, banner_final_score() function, RESTRICTIVE DENY RLS.

**How to apply:**

1. **2×2 rendering matrix** (96dp GST × 80dp tab × icon/illustration) — all variants unified whole-card tap, NO pill CTA anywhere, illustration always right-rail with 96×96 / 80×80 `BoxFit.contain` clamps. Dismiss × on TAB only; GST has no dismiss.

2. **Bug 2 fix** requires BOTH gate-success-only counting AND `bannerRenderedTutorialIdsProvider` (keepAlive Set<int>) — currently-displayed card registers on mount, unregisters on dispose; engagement filter subtracts rendered IDs from exclude set. Single fix alone is incomplete.

3. **TextScaler.clamp(1.0, 1.2)** + `FittedBox(scaleDown)` — 20% corridor tightened from 1.3 per user. Pure `clamp(1.0, 1.0)` violates WCAG 1.4.4; raw `fontSize/textScaleFactor` division inverts at deviceScale<1.0 (GOTCHA-S03).

4. **3-way CHECK contract** — JSON fixture at `supabase/tests/fixtures/cta_type_values.json` is single source; pgTAP validates SQL ↔ JSON; separate Dart test validates const ↔ JSON (pgTAP can't read Dart).

5. **Migration numbering** via `make migration-new` (sentinel-continuous from 000266 → 000267+); plan-specified 000281/000282 would fail validator.

6. **KV purge sequence** post-migration: `watch-state:*` (5min TTL) + `banner:v3:*` (30min TTL); else users stay excluded after data cleanup.

7. **Pre-launch big-bang rollout** — app isn't live on Play Store yet, so `flag:gst_v2_enabled` binary killswitch is sufficient. Dropped rollout percent KV + hash scaffolding.

8. **Illustrations direct from R2** via `R2_PUBLIC_PREFIX` (public URLs, edge-cached). Removed Worker's `/v1/banners/illustration/:key` proxy endpoint.

9. **Open-closed architecture**: `BannerShell` (container + whole-card tap + matrix routing) + `BannerContent.iconVariant` / `BannerContent.illustrationVariant` — new variants add without modifying existing code.

10. **Emergency banners** (`banner_type='emergency'`) bypass engagement cap + session dismiss + placement-category affinity; still respect auth + min_build_number.

**Key files (52 total, 28 Flutter + 10 migrations + 8 pgTAP tests + 4 Worker + docs + CLAUDE.md updates):**
- New providers: `banner_cta_dispatcher.dart`, `banner_engagement_reconciler.dart`, `banner_rendered_ids_provider.dart`, `banner_impression_batch_provider.dart`, `banner_offline_cache_provider.dart`
- New widgets: `banner_shell.dart`, `banner_content_icon.dart`, `banner_content_illustration.dart`, `banner_dismiss_button.dart`
- New utils: `banner_gradient_registry.dart`, `banner_icon_registry.dart`, `banner_clamp_constants.dart`, `banner_press_feedback.dart`
- CLAUDE.md updates: `lib/billbook/tutorials/banner/`, `microservices/tutorials/`, `test/billbook/tutorials/banner/` (new file)

**Status:** Plan at Round 10; Ultraplan refinement pass pending before ExitPlanMode.
