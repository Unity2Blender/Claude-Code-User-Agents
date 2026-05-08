---
name: Worker rotation logic must respect v5 RETURNS TABLE shape (no banner_id, salience, sentinel exposed)
description: get_banner_recommendation_v5 returns 15 cols projecting tutorial fields. Worker cannot weight by salience or log real banner_id. Salience-aware rotation must live in SQL ORDER BY, OR a separate metadata RPC must be added.
type: feedback
originSessionId: a077cb79-f425-4b9b-b98a-8852dbb40b6b
---
`get_banner_recommendation_v5` (000404 + 000442) RETURNS 15 columns, all projected from `tutorials_reels` (id = tutorial id per 000379). It does NOT expose:
- `tutorial_banners.id` (banner_id)
- `tutorial_banner_placements.placement_salience`
- `tutorial_banner_placements.is_sentinel_at_placement`

**Implication:** Worker code reading the v5 response cannot:
- Weight rotation by per-banner salience (no salience field).
- Log "banner_id" honestly (only tutorial_id is available).
- Treat sentinel rows specially (no flag).

**How to apply:**
- Salience-aware ordering MUST live in SQL inside v5 (ORDER BY salience DESC, sentinel DESC, t.id ASC).
- Worker rotation is purely positional: deterministic shuffle by `hash(uid+placement+hour_bucket)` over the already-ranked array. No salience math on Worker side.
- Worker per-serve logging uses `tutorial_id` field name (not `banner_id`) unless a parallel metadata RPC is introduced.
- If the spec needs Worker to see banner_id/salience/sentinel, the answer is a NEW Worker-internal RPC (e.g., `get_banner_pool_meta_v1`) — NOT modifying v5 RETURNS TABLE (that triggers vN+1 + Flutter contract change).
- 2026-05-05 plan Rev 2 audit caught this conflation; fix is "rotation lives in SQL ORDER BY + Worker positional shuffle, log tutorial_id only".
