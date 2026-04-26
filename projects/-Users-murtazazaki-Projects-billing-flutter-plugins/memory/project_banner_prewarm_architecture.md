---
name: Banner Pre-Warm architecture (GST cache-hit + reel-first)
description: 16-decision system shipped 2026-04-16 — post-frame prewarm + auth-transition prewarm + reel-first intent filter + 128dp compact GST banner
type: project
status: contextual-history
originSessionId: c96fe297-452d-494c-9bd9-54d920a7b117
---

**Kernel status:** Historical/contextual prewarm implementation record. Use it when debugging Banner Pre-Warm. Do not generalize its killable flag design into future UI work. Current active doctrine is `MEMORY.md`: no UI runtime flags unless the user explicitly asks.

# Banner Pre-Warm System (shipped 2026-04-16, commit 2f61a135)

Plan: `.claude/plans/nested-whistling-church.md` (16 decisions + 6-agent scrutiny audit)

## Why

- GST result page paid 300–900ms cold Worker fetch → banner impressions lost.
- Banner at 220dp was too tall vs GstBreakdownCard + crowded breathing space.
- GST-surface CTAs deep-linked to features ("Log in to Record Payment") — user wanted reel playback (90% case).

## Architecture (3 bundled concerns)

### 1. Prewarm orchestration
- `BannerPrewarmNotifier` (`@Riverpod(keepAlive: true) class`) — `run(trigger:, currentBuildNumber:)`.
- Dedup: `Completer<BannerPrewarmResult>? _inFlight` — second `run()` shares the first's Future across post-frame + auth-transition triggers.
- Triggers:
  * D1 post-frame from `schedulePostFrameWarmups` (apps/gst_calculator/lib/app/deferred_init.dart)
  * D7 auth-transition from `AuthTransitionNotifier.runTransition` after `validatingClaims`
- Fan-out via `BannerFanoutResolver.resolve(...)` pure function. Anon users get `{gst}` only; signed-in + has-firm + domain data unlock tab-bills/items/parties.
- 3s summary timeout, 5s per-placement, 2s backendReady — all fail-silent.
- Killable via `flag:banner_prewarm_enabled` KV flag (default TRUE).

### 2. Lightweight summary RPC
- `public.get_user_lightweight_summary()` (migration 000278): SECURITY DEFINER STABLE PARALLEL SAFE, 2s statement_timeout, 6 EXISTS subqueries returning `{has_firm, has_items, has_parties, has_invoices, has_payments, has_ocr_scans}`.
- **CRITICAL**: Does NOT call `public.user_billing_id()` — reads JWT claims inline to dodge the STABLE session-caching bug (see `project_user_billing_id_stable_caching.md`).
- Client-side `userLightweightSummaryProvider` short-circuits anon BEFORE `postgrestProvider` (prevents GOTCHA-030 AsyncError poisoning the keepAlive cache).
- Invalidation via `invalidateUserLightweightSummary(Ref)` helper — callers invoke after mutation success.

### 3. Reel-first CTA filter (D4)
- `banner_placement_config.intents = ['reel-watch']` for gst + gst_activated (migration 000279).
- Requires `chk_tutorials_intent` CHECK-ALLOWLIST widening + TS mirror (`schema-constants.ts INTENT_VALUES`) + pgTAP contract test update — all same commit.
- Currently returns empty pool until content-ops seeds tutorials with `intent='reel-watch'` + `cta_config.type='reel-watch'`. `gst_v2_enabled` flag is dark-launched (default FALSE) so prod users see DynamicFooter fallback, zero regression.

## GST compact layout
- `TutorialBannerCard(height: 128, useTonalCta: true)` on GST result page.
- 128dp = DynamicFooter State A/B (~132dp) — zero cross-fade jump for ~60% of sessions on sign-in / set-up-firm paths. `height: 96` was initially chosen but UI audit proved content overflows at 1.3× textScaler in Mode B (BLOCKER UI-B1).
- `useTonalCta` → `FilledButton.tonalIcon` → avoids primary-emphasis competition with Total hero (HIER-001).
- Dark-mode surface: `surfaceContainerHigh` (not surfaceContainerLow) against GstBreakdownCard's `surfaceContainer` background.
- `Padding(top: 12, bottom: 8)` around banner/fallback slot — user's explicit "breathing space" instruction (mentioned twice).
- AnimatedSwitcher duration 300→200ms since heights now match.

## Telemetry
- 4 Firebase Analytics events correlated by `prewarm_session_id`: `banner_prewarm_started`, `banner_prewarm_ready`, `banner_served_from_prewarm` (widget-side), `banner_prewarm_stale`.
- `BannerPrewarmTelemetry` sink wraps `bannerPrewarmAnalyticsSinkProvider` (overridden in `provider_overrides.dart`).
- Hit-rate target: >70% `served / (served + stale{!=disabled})` per placement after 1 week.

## Rollout / Rollback (5-step coordinated, audit DO5)

Rollback: `wrangler kv key put flag:banner_prewarm_enabled false --binding TUTORIAL_CACHE --remote` — **MUST include `--remote`** per CP-26 (wrangler v4 default writes to local miniflare otherwise).

Propagation: ≤3 min cold-start (KV ~60s + edge ~60s + Cache-Control max-age=60), NOT 60s as initial plan claimed.

## Known deferred work

- **X-Prewarm-Origin: 1 HTTP header** (audit DO6): Worker-side handling is wired (`banner.ts` `maybeLogBannerRoute` logs the flag), but client currently does NOT send the header. Server-side hit-rate cross-check is nice-to-have; client-side analytics suffice for measurement. Follow-up if needed.
- **Content-ops runbook** (migration 000279 comment): need to seed tutorials with `intent='reel-watch'` + `cta_config.type='reel-watch'` before flipping `gst_v2_enabled=true` — else GST placement serves empty pool.
- **Flutter tests 3–10 from Phase 8**: MATPU sentinel, fail-silent, render-mid-prewarm timing (fake_async), killswitch OFF, mutation rollback non-invalidation, summary-error fan-out, 128dp widget test — skeleton is the BannerFanoutResolver test (13 tests shipped in `banner_fanout_resolver_test.dart`). Follow-up PR.

## How to apply

Before re-invoking prewarm or touching banner surfaces in future sessions:
1. Check `.claude/plans/nested-whistling-church.md` for the 16 locked decisions — don't re-question them without user re-interview.
2. The 3 concerns (prewarm + compact + reel-first) are SEPARABLE for rollback: flipping `banner_prewarm_enabled=false` only kills prewarm; `gst_v2_enabled=false` kills the new banner rendering entirely. Use the narrower flag first.
3. `userLightweightSummaryProvider` is keepAlive — invalidate via the `user_summary_invalidator.dart` helper after mutations that flip `has_X` bits (invoice create, item create, party create, payment, OCR convert).
