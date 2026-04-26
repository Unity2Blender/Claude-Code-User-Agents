---
name: Tutorial Engine v5 locked decisions
description: 38 decisions from Reels hardening + Worker activation + MATPU lockdown audit (2026-04-08), covering config, auth, views, banners, MATPU, fail-closed UX, kill switches, FCM parity
type: project
---

38 decisions locked during Tutorial Engine v5 plan audit (2026-04-08):

**Config (D1-D2):** flutter_dotenv with LOCAL/PROD split (both `tutorials.gstapps.com` initially). TutorialEngineConfig singleton follows OcrConfig pattern.

**Auth (D7, D18):** Remove `requireBillingContext()` gate AND MATPU guard on Reels entirely. Anonymous users browse freely. Worker handles auth via Firebase JWT; service_role key for all Supabase operations.

**Views (D9-D10, D12, D16):** Aggregate counters (`opened_count`, `completed_count`) on tutorials table for ALL users. Per-user tracking in `tutorial_views` (signed-in only). Worker POST /views: one endpoint with internal branching.

**Banners (D1, D4-D5, D8, D17):** Separate `BannerDataSource` interface. Banner RPC gets `p_context JSONB DEFAULT NULL`. Context via query params on GET.

**Seen Provider (D6, D11):** AsyncNotifier<Set<int>>. Full migration with `.when()`.

**Platform (D15):** Android-only Videos tab gate.

**DB (D3):** Bundle `updated_at` fix + aggregate counters in migration 000230.

**Scope (D13-D14):** Worker changes in scope. Phase 9 UX enhancements (all 5 done).

---

### Hardening Decisions (D19-D38, 2026-04-08)

**D19 Fail-closed UX:** Lazy fail-on-tap. Grid renders hardcoded cards even when Worker is down.

**D20 Cooldown:** Worker KV cooldown per-UID per-tutorial, 30s window. Key: `cooldown:{uid}:{tutorial_id}`.

**D21 Kill switches:** KV flags `flag:reads_enabled` and `flag:writes_enabled`. Toggle via `wrangler kv:key put`. 503 + SERVICE_DISABLED when off.

**D22 FCM parity:** Shared `ReelsLauncher` utility. Both VideosHomeView and FcmReelsLauncherPage call it.

**D23 Transport race:** FutureProvider awaiting `backendReadyProvider`. No more sync keepAlive race.

**D24 CTA behavior:** Auth (AuthTransitionSheet full pipeline) + Update (immediate InAppUpdate) for v5. Navigate/paywall stay as stubs.

**D25 Cache key:** build_number + surfaceId in playlist cache key.

**D26 MATPU lockdown (CRITICAL):** REVOKE ALL from anon/authenticated on ALL tutorial RPCs. Remove SupabaseTutorialDataSource entirely. Worker-only model is non-negotiable. Client NEVER touches Supabase for tutorials/banners.

**D27 Error vs empty:** Throw `TutorialServiceUnavailableException` on HTTP errors. Honest UX: distinguish Worker-down from no-content.

**D28 Auth CTA:** `AuthTransitionSheet.show()` with `kBillingAuthSteps` (full pipeline).

**D29 Update CTA:** `performImmediateUpdate()` from existing `updateUxProvider`. Fallback to Play Store URL.

**D30 Banner cache:** Skip context in cache key until SQL uses it. Include `limit`.

**D31 Kill switch HTTP:** 503 + JSON body `{ code: 'SERVICE_DISABLED' }`.

**D32 Interface:** Keep `TutorialDataSource` interface for testability (single prod impl + mock).

**D33 Cooldown scope:** Per-UID per-tutorial. User can view multiple tutorials in 30s.

**D34 Grid error UX:** Hardcoded cards + subtle error hint ("Couldn't refresh — tap to retry").

**D35 Banner MATPU:** Remove `SupabaseBannerDataSource` too. Worker-only.

**D36 Legacy RPC:** DROP `register_tutorial_view` entirely. Only `register_tutorial_view_admin` via Worker.

**D37 Migration fix:** New migration 000232 (000231 already taken).

**D38 FCM platform:** Android-only FCM Reels. Suppress on non-Android.

**Why:** 32-finding audit revealed MATPU violations, counter gaming, transport race, missing kill switches, FCM parity gap. MATPU is existential risk (Supabase MAU billing).

**How to apply:** These decisions supersede D1-D18 where conflicting. Migration 000232 enforces MATPU lockdown at DB level.
