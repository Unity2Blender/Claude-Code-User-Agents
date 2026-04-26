---
name: Internal admin Flutter web app under apps/tutorials_admin
description: Sibling to apps/gst_calculator and apps/billing-web — internal-tooling Flutter web app, same Supabase project, admin role gate; deferred follow-up after banner↔reel 1:1 retrofit lands
type: project
originSessionId: f0d52fc2-4fa9-4b08-b682-f3036421b900
---
Operator clarified 2026-04-25 that the "schema room / super-admin dashboard" is a real internal tooling surface, NOT generic community marketing.

**Path:** `apps/tutorials_admin/` — sibling to `apps/gst_calculator/` and `apps/billing-web/`.

**Type:** Flutter web app, internal-use-only. Gated by Firebase Auth JWT role claim (`role='admin'`). Same Supabase project, same database, same RLS — admin role is the only diff vs customer-facing apps.

**Scope (operator-stated capabilities):**
- Banner organization visualizer — categories × placements × banner rows; R2 illustration thumbnails; 2D matrix.
- Reels graph visualizer — `tutorials_reels` × `tutorial_reel_edges` × `tutorial_source_edges` as force-directed graph.
- Banner ↔ reel 1:1 audit — orphans, archived-reel pointers, FK-RESTRICT violations.
- Schema change preview/staging — paste candidate migration → run on Supabase preview branch → diff schema before/after.
- Reels script director integration — surface "scripts authored vs reels in DB" so the script director sees promotion gaps.

**Architectural principles:** Riverpod 3.2; admin RLS-respecting reads (no new RPCs unless aggregate not in Worker); `graphview` or `flutter_force_directed_graph` for graph; `fl_chart` for time-series; reads `supabase db dump --linked --schema=public` cached output (per MIGRATE-020) for current-schema view; deploys to Cloudflare Pages on `tutorials-admin.gstapps.com` (or `/admin` path with auth gate).

**Out of scope for v1:** write paths (banner editor, reel uploader). Read-only inspection first.

**Why:** Migrations are incremental — inferring current banner/reel/edge state by grepping migration files is O(N) every time; misses out-of-band Studio edits; brittle against compat-view drops. The dashboard reads live schema dump + DB tables to answer content-ops questions in seconds.

**How to apply:** File as separate plan after `tutorials-banners-reels-edges-tender-grove.md` Phase 1–3 land. Reference rule MIGRATE-020 (db dump live-state inference) and MIGRATE-019 (iterate db push --linked --dry-run) from `design-postgres-tables` skill.

**Companion:** Reels script director skill (`reel-script-director`) already touches the schema; admin app surfaces script-vs-reel gaps it can't see today.
