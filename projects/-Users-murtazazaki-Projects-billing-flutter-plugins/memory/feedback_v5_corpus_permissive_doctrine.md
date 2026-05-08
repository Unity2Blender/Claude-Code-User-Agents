---
name: Banner serving predicates align with v5 corpus, NOT is_reels_eligible
description: Pinned by `banner_v5_corpus_stays_permissive_test.sql`. Banner pool/depth/breadth predicates use is_media_ready=TRUE (and active+published+production), never is_reels_eligible.
type: feedback
originSessionId: a077cb79-f425-4b9b-b98a-8852dbb40b6b
---
`get_banner_recommendation_v5` serves rows where `is_reels_eligible = FALSE` provided `is_media_ready = TRUE`. The pgTAP file `supabase/tests/database/critical/banner_v5_corpus_stays_permissive_test.sql:6` makes this an explicit critical-tier invariant.

**Why:** Banners and reels are different surfaces. Banners want max corpus reach; reels (Videos tab) wants curated reels-eligible content. Filtering banners by `is_reels_eligible` would over-exclude valid wizard-step tutorials.

**How to apply:**
- Banner-pool backfill, source-edge seeding, depth-invariant tests, and any v5-aligned RPC reuse the same predicate set:
  - `tutorial_banners.is_active = TRUE AND status = 'published'` (and not retired)
  - `tutorials_reels.is_active = TRUE AND publication_status = 'production' AND is_media_ready = TRUE`
- Do NOT add `is_reels_eligible = TRUE` to banner-side queries.
- The Reels-tab corpus query (`get_video_playlist`, `search_reels_v1`) is the correct place for `is_reels_eligible`.
- 2026-05-05 plan Rev 2 incorrectly applied `is_reels_eligible` to banner backfill; caught at audit via grep of the permissive test.
