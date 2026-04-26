---
name: Placeholder reels are test vehicles for CTA correctness
description: Operator philosophy — placeholder tutorial rows exist to validate banner→CTA→intent→swipe-graph BEFORE R2 video URL is attached; only the video URL is pluggable
type: feedback
originSessionId: 1636fa18-93d5-4e42-b373-51c0d539de25
---
**Rule:** Placeholder tutorial rows (`tutorials_reels.publication_status='placeholder'`) are test vehicles for app-layer CTA dispatch correctness. They can legitimately exist WITHOUT `video_url_mp4` (no video authored yet). The banner + CTA + reel-swipe graph MUST work end-to-end before any video is attached.

**Why:** Operator wants to validate that (banner recommendation → reel dispatch → intent-within-app → swipe graph traversal) is wired correctly BEFORE committing Cloudflare R2 upload. Stated during ADR-007 brainstorming interview 2026-04-22: "the flow and recommendation of that banner, and then to be able to watch that video and click on the action which leads you towards the intent within the app layer. Apart from that, within the app layer, when you swipe up and down, what the graph and edges are supposed to traverse. So these are the few things which we have to be aware of."

**How to apply:**
1. Do NOT tighten placeholder-row CHECK constraints to require `video_url_mp4` or other media-readiness fields. ADR-007 tightened ONLY `category` + `cta_config` (browse/open contract) — explicitly rejected media-readiness gating (LD-14).
2. When placeholder banner is tapped, `ReelVideoCard` routes to `ReelPlaceholderCard` for null video URL (LD-10 placeholder-open = success, not error).
3. Only pluggable thing per tutorial row: the `video_url_mp4`. Banner recommendation contract + CTA dispatch + swipe graph edges are STABLE.
4. `min_build_number` authored manually per tutorial at 20-50 initial reels + 2-3/day ongoing — operator chose manual discipline over runtime infra (LD-4 / YAGNI-005).

**Source:** ADR-007 LD-9/LD-10/LD-14. Placeholder rows are the designed water-test surface that ADR-006's "turn on the water" intentionally exposed.
