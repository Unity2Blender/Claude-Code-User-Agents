---
name: Banner SRP — version gating lives on the reel, not the banner
description: Refinement of BANNER-SRP-001 — `min_build_number` is a reel-layer concern; banner stays SRP-pure by not carrying version checks
type: feedback
originSessionId: 1636fa18-93d5-4e42-b373-51c0d539de25
---
**Rule:** Banner's single responsibility is to open a tutorial reel by `tutorial_id`. Version gating (`min_build_number`), entitlement, prerequisite, connectivity all belong to the REEL's own CTA chain — NEVER hoisted into the banner.

**Why:** Operator stated during ADR-007 brainstorming 2026-04-22: "banner single responsibility principle does not concern whether this banner, whatever you rotate as the placeholder or any actual banner, is within its scope or not. There would be a minimum build number. With respect to that, the banner can compare that, 'Hey, I'm in so and so build, this particular reel which will be opening.' Okay. So separation of concern, again, I'm asserting that it is for reels to know what minimum version is."

**Why (infra consequence):** YAGNI-005 vetoed `banner_runtime_config` table / dashboard knob. `tutorials_reels.min_build_number` column ALREADY exists — manual per-reel authoring is sufficient for 20-50 initial reels + 2-3/day ongoing (operator's stated cadence).

**How to apply:**
1. Do NOT introduce banner-side `min_build_override` fields, runtime config tables, or dashboard knobs. Pubspec `version` + Codemagic auto-increment is enough.
2. When banner renders, it does NOT check build number. The REEL destination (ReelVideoCard + ReelFooter's CTA resolver) does.
3. Advisory pgTAP test at `supabase/tests/database/advisory/published_reels_min_build_number_reminder_test.sql` emits `diag()` list of production reels missing `min_build_number` — content-ops review, not a hard gate.
4. ADR-007 LD-4 is the canonical reference for this refinement. `BANNER-SRP-001` test guards the 2-field `BannerLaunchCallbacks` interface.

**Source:** ADR-007 LD-4. Refines `project_banner_launch_controller_srp.md` (existing).
