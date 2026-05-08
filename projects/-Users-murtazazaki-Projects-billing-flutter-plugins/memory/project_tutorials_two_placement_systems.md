---
name: Tutorials has two parallel placement systems
description: banner_placement_config (RPC serving) vs tutorial_sources (graph traversal). Modifications affecting placements MUST touch BOTH or surfaces drift.
type: project
originSessionId: a077cb79-f425-4b9b-b98a-8852dbb40b6b
---
Tutorials runs **two parallel placement registries** that must be modified together when placements change:

**System 1 — Banner SERVING:**
- `banner_placement_config` (000247) holds the placement allowlist (5 seed rows: gst, gst_activated, tab-bills, tab-items, tab-parties).
- `tutorial_banner_placements` (000410) is the junction `(banner_id, placement_id)` with `placement_salience` (000436), `is_sentinel_at_placement`.
- `get_banner_recommendation_v5` (000404/000442) reads from `tutorial_banners` + `tutorial_banner_placements` + `banner_placement_config`.
- This is the live serving path used by Worker `GET /v1/tutorials/banner`.

**System 2 — Graph TRAVERSAL:**
- `tutorial_sources` (000304) is a unified registry with `source_type ∈ {banner_placement, reels_category}`. Prod has 6 banner_placement rows (gst, tab-bills, tab-items, tab-parties, gst_activated, tab-reels) + 7 reels_category rows.
- `tutorial_source_edges` (000305) maps `source_id → target_tutorial_id` with `priority`, `is_primary`. Exact column names: `id, source_id, target_tutorial_id, priority, is_primary, is_active, ends_at, starts_at, metadata, created_at, updated_at` (NOT `tutorial_id` or `ord`).
- `get_tutorial_for_source` and `get_reel_sequence_next` (000307) walk this graph.
- Drives `tutorials_admin` Full Reel Graph cockpit and reel-to-reel recommendations.

**Why:** First system answers "what banner shows on placement X?"; second answers "which reels are reachable from placement X?". Conflating them produces wrong fixes — adding source_edges does not increase banner candidate depth, and dropping a placement only from `banner_placement_config` leaves a stale source in the admin graph.

**How to apply:**
- Banner candidate-depth fixes go to `tutorial_banner_placements` junction rows.
- Orphan/reachability fixes go to `tutorial_source_edges` rows.
- Renaming/removing a placement requires DELETE from BOTH `banner_placement_config` AND `tutorial_sources` (plus cascading rows in their respective edge/junction tables).
- The 2026-05-05 GST consolidation plan amendment cycle was triggered by missing System 2 in the initial plan.
