---
name: External Twitter/LinkedIn/Instagram/GitHub publishing is reels-engine-bound, not Supabase community marketing
description: Don't propose generic Supabase community touch-points (meetups, generic posts) — external publishing flows through the tutorials reels-engine itself; every external post is a reel
type: feedback
originSessionId: f0d52fc2-4fa9-4b08-b682-f3036421b900
---
Operator clarified 2026-04-25: external publishing on Twitter / LinkedIn / Instagram / GitHub is **inbound-internal marketing only** — every external post flows through the tutorials microservice (tutorials.gstapps.com edges and reels). It is NOT generic Supabase community presence.

**Why:** During Phase 4 design of the banner↔reel retrofit audit, I misread "maintain this dashboard for Supabase every three months" as a request for community marketing cadence (Supabase meetups, social touch-points). The operator's actual intent is the tutorials engine itself produces the external posts; the "internal" dashboard is the schema/admin surface for staging that content.

**How to apply:**
- Don't propose `/schedule` recurring tasks for "post to Twitter about Supabase architecture" or "host quarterly Supabase meetup."
- DO propose internal admin app capabilities that stage / preview / publish reels as social posts.
- When operator says "marketing channel," reach for the reels-engine first; only branch to community work if explicitly named.
- Quarterly cadence applies to: schema doc audit, super-admin dashboard refresh, reels content review — NOT external community work.
