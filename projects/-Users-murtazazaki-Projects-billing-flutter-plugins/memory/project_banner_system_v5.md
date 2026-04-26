---
name: Banner System v5 — locked architecture + Phase A + A2 shipped
description: v5 unifies Channel 3 (GST) + Channel 5 (Bills/Items/Parties tabs) under one tokenized renderer with optional R2 illustration overlay. Phase A DB layer shipped 88914b52. Phase A2 Worker layer shipped 0a477c38 on 2026-04-13 — 3 new routes + dual-layer killswitch + 119 Jest tests.
type: project
status: contextual-history
originSessionId: 336ba39e-a32b-4430-a1b5-746b61131666
---

**Kernel status:** Historical/contextual Banner v5 record. Do not generalize its KV flag or UI coexistence strategy into new UI work. Current active doctrine is `MEMORY.md`: no UI runtime flags unless the user explicitly asks.

Plan file: `/Users/murtazazaki/.claude/plans/moonlit-brewing-sonnet.md` (approved 2026-04-12, 105 directives).
Phase A plan: `/Users/murtazazaki/.claude/plans/cached-sauteeing-sun.md` (approved + implemented 2026-04-13).

**Why:** SLC-minded v5 (Simple-Lovable-Complete, not MVP). Unifies banner engine for long-run correctness. FCM push (Channel 4) explicitly excluded — separate future plan.

**How to apply:** Before any code on banner/reels/tutorials surfaces, re-read the plan's directive ledger (L1–L105). Critical locked decisions that must NOT be reopened:
- L55: engaged-watch = `(≥50% AND cta_tap_count≥2) OR (≥75% AND ≥1) OR ≥99%`
- L62/L91: dismiss = app-runtime `StateProvider<Set<int>>`, shared across dual hosts via ONE `ProviderScope`, widget filters locally (LazyIndexedStack safety)
- L67/L84/L90: ONE tokenized renderer with optional `illustration_overlay JSONB` — NO `render_variant` enum
- L85/L97: Mode A/B/C auto-switch at layout time with 8lp hysteresis
- L88/L96: Worker-fronted `GET /v1/banners/illustration/:key` route for sub-60s R2 purge (vs 14d client cache)
- L100: GOTCHA-035 authStateNotifier dispose fix (SHIPPED commit f4ab0749)
- L89/L103: Bills tickers = `To Collect` + `Sales Amount` (filter-aware, static label "Sales")
- L65/L73: 5 KV killswitches — `flag:banner_global_enabled` (master) + 4 per-placement
- C1/L69: `register_cta_tap` uses INSERT…ON CONFLICT UPSERT (lost-update safe)
- C2/L70: banner cache-key hashes from resolved config, not request params; TTL invariant banner(30m) < config(1h)
- C3/L71: cta-tap response body returns new counters for optimistic apply
- L64: session-only dismiss forever (no 30d persistence ever)

**Migration ledger (renumbered 000245–000252 — HEAD shifted +2 vs parent plan):** 8 forward migrations, no rollback files (session-chosen deviation P-A-01).

**Phased rollout:** A (DB+Worker+Flutter — 3 sub-phases), B (GST refactor + Bills), C (Items), D (Parties+GA).

**Phase A DB layer — SHIPPED 2026-04-13 (commit 88914b52):**
- 000245: tutorial_views.cta_tap_count + covering index (partial WHERE reverted — PG requires IMMUTABLE predicates for NOW()/CURRENT_DATE).
- 000246: tutorials.intent (4-value CHECK) + category-driven seed-classify. Actual row count = 19 (NOT 16 as parent plan assumed; 000224 added 3 reels).
- 000247: banner_placement_config table + 5 seed placements. RLS on, no policies (service_role bypass).
- 000248: tutorial_banners.placement_id FK (CASCADE/RESTRICT) + illustration_overlay JSONB shape CHECK (uses `r2_key` per G17, NOT `r2_url`).
- 000249: chk_tutorial_category 8→15 (7 new: e_invoicing, compliance, leads, manufacturing, desktop, troubleshooting, retail_pos). Parent plan said 5; session expanded to 7.
- 000250: get_banner_recommendation v3 — 8 IN params, 15 OUT cols. `id BIGINT` stays first (000233 already renamed; parent plan G15 "mismatch" was stale).
- 000251: get_user_tutorial_views v2 — 5 return cols, tutorial_id first (additive).
- 000252: register_cta_tap UPSERT.

**Ground-truth corrections applied in Phase A session (G15–G19):** v2 RPC already returns `id` first (000233 fixed it); 19 tutorials not 16; `r2_key` not `r2_url`; no pre-existing `tutorial_banners.placement_id` column to migrate from; rollback strategy = forward-only house style, NOT `supabase/rollbacks/` directory.

**Phase A test coverage (all green):** 9 new pgTAP files + 3 edited = 118 assertions. CHECK-ALLOWLIST-001 drift-guard in critical/tutorial_enum_allowlists_test.sql mirrors `microservices/tutorials/src/schema-constants.ts` INTENT_VALUES + TUTORIAL_CATEGORIES.

**Phase A2 Worker layer — SHIPPED 2026-04-13 (commit 0a477c38):**
Plan file: `/Users/murtazazaki/.claude/plans/hazy-plotting-wilkes.md`. devops-architect audit integrated — 12 findings, 7 amendments (A2D-001/004/007/008/010/011/012) + 7 new directives (A2D-013–019).
- NEW routes: `cta-tap.ts` (Native Rate Limit 60/60s/uid via CTA_TAP_RATE binding, A2D-004 3-branch auth), `banner-illustration.ts` (public R2 CDN URL proxy via `R2_PUBLIC_PREFIX` var — no bucket binding), `schema-drift.ts` (Bearer-gated, reuses CACHE_PURGE_TOKEN).
- NEW service: `placementConfigService.ts` — KV 1h cache, SHA-1 cat_hash/intent_hash from resolved config row (C2/L70), `remapPlacement()` journey-stage helper (A2D-002 — runs before config lookup).
- banner.ts v3 refactor: placement+exclude_ids (cap 200, A2D-003), 8-param RPC, sampled logs (A2D-011 10%/100%).
- killSwitch dual-layer: `isPlacementFlagEnabled` with INVERTED default (missing=FALSE per audit C-2 deploy-race fix), `requireBannerPlacementEnabled` short-circuits `c.json([])` (A2D-001, audit H-4 — handler decoupled).
- cacheService: do-while cursor loop with `list_complete` gate (audit H-1 fixed wrong for-await syntax), `kv_write` log (M2/L77).
- Makefile: `deploy-{test,prod}-full` composites (A2D-014), `set-kv-flags-{test,prod}`, `purge-watch-state-{test,prod}` (A2D-012 PRE-deploy), `admin-purge PATTERN=... ENV=...` with retry×3 (M1/L76), `lint-wrangler` gate (A2D-013), 2MB bundle-size gate in deploy.sh (A2D-016).
- Rollback: `supabase/rollbacks/rollback_000252.sql` (A2D-009 fills H4/L75).
- Test count: 119 green across 14 suites (was 39).

**Key Phase A2 implementation gotchas worth remembering:**
- KV `list()` returns single page — NOT an async iterator. Plan initially had wrong `for await` syntax; fixed to do-while with `list_complete` gate.
- `caches.default.delete(new Request(url))` is PER-COLO — doesn't purge zone-wide edge cache. Cloudflare Cache Purge API would need CF_ZONE_ID + CACHE_PURGE_API_TOKEN; **deferred to Phase B** (L94 key-suffix-bump workflow accepted for v5).
- Placement flags use INVERTED default (missing=FALSE) vs global/reads/writes (missing=TRUE) — prevents deploy-window race where flags absent briefly after rollout.
- Hono middleware short-circuit via `return c.json([])` from middleware itself is cleaner than `c.set('flag')` + handler check — removes coupling.
- CF Native Rate Limit binding `[[ratelimits]]` added under `unsafe.bindings` (still under `unsafe` namespace as of wrangler CLI 4.7).

**Next sessions:**
- **Phase A3 (Flutter banner module, ~2 sessions):** Create `lib/billbook/tutorials/banner/` with 10 files per L99/L101/L102. `banner_layout_resolver.dart` pure fn, `BannerCard` thin ConsumerWidget, `banner_provider.dart` `@riverpod` autoDispose + conditional keepAlive (L101). ~130 unit tests + 30 goldens + MATPU sentinel.
- **Phase B:** Retire DynamicFooter, inject BannerCard(placement: tab_bills), flip flag:gst_v2_enabled, ticker reduction (L59).
