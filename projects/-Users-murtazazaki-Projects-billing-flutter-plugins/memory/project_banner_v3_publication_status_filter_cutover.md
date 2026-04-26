---
name: Banner v3 publication_status filter cutover — 2026-04-22 outcome
description: Post-cutover state after 000351 publication_status filter + probe server-side fix; GST served via Layer 2, tab placements content-starved
type: project
originSessionId: e2f8e44b-274b-414f-a32e-6a5f6b0e99b4
---
**Cutover commit:** `c89a15ff` deployed 2026-04-22. Included probe fix `ca60cf6e` (server-side predicate before `.limit(5)`).

**KV purges applied:**
- `make admin-purge-banner-all ENV=production` → banner:v3:*=0 (cold), placement-config:*=5
- `make admin-purge PATTERN='playlist:*'` → deleted 3 keys
- `make admin-purge PATTERN='categories'` → deleted 1 key

**Probe state post-cutover:**
- `/v1/admin/health-data-sentinels` → `gst_sentinel.healthy:true, actual:1` ✅
- `/health/data` → `content_degraded` (gst_activated, tab-bills, tab-items empty; gst + tab-parties served)
- Workers Logs 5-min tail: zero `gst_invariant_violated`

**Content-layer reality exposed by 000351 filter:**

Only tutorial id=31 has `publication_status='production'` in the gst banner pool. Other 16 gst banners map to tutorials with `publication_status='placeholder'` — correctly filtered. So the primary pool reduces to the sentinel.

BUT the sentinel's intent (`'acquisition'`) does NOT match the gst placement-config intents array (`['reel-watch']`). So v3 RPC returns `[]` for cohort-correct calls with the placement-config intent filter applied.

**Why LD-34 always-on still holds:** The user-facing `/v1/tutorials/banner` route hits Layer 2 fallback (`banner.ts:558-660`) when v3 returns empty. Layer 2 reads the sentinel directly from `tutorial_banners` with `is_sentinel=true` predicate, bypassing placement-config intent filter. `/health/data` aggregate probe for `gst` uses the same fall-through logic → reports `served`. The cohort-correct `/health/banner/gst?cohort=anon` probe calls v3 directly → reports `empty_pool`.

**Content-ops follow-ups (NOT blocking, docs-only for now):**
1. Either update `tutorials_reels.intent` for id=31 (`'acquisition'` → `'reel-watch'`) OR extend `banner_placement_config.gst/gst_activated.intents` to include `'acquisition'`. Removes Layer 2 dependency for the happy path.
2. Promote placeholder tutorials for tab-bills (`invoicing/payments/e_invoicing/compliance` × `activation/retention`), tab-items (`inventory/manufacturing`) from `placeholder` → `production` per D7 14-day SLA in `docs/runbooks/tutorials-content-gaps.md`.

**Rollback not triggered.** The empty-tab surfaces render as silent collapse (D6), not crashes. No new Crashlytics error class. User-visible GST banner intact.

**Where it's documented:** `docs/runbooks/banner-pool-empty-debug.md` "2026-04-22 — Post-cutover cutover record + content-ops gaps" section.
