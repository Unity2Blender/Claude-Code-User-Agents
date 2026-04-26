---
name: Reels Tab Cloudflare Workers Migration (HIGH PRIORITY)
description: Reels data must migrate from direct Supabase to Cloudflare Workers for anonymous/production access — blocks production launch
type: project
---

Reels tab data layer must migrate from direct Supabase `postgrestProvider` to Cloudflare Workers for production.

**Why:** In production, anonymous (non-signed-in) users must access the Videos/Reels tab. Current MATPU guard blocks anonymous access via Supabase. Cloudflare Workers will serve as the API layer, removing the auth requirement for public-read tutorial data. This is a production launch blocker.

**How to apply:**
- Keep MATPU guard on `tutorialCategorySummaryProvider` during testing phase (current)
- Design the provider/data layer with an abstraction that can swap Supabase for CF Workers without UI changes
- When implementing, create a CF Worker endpoint that serves `get_category_video_counts()` and `get_video_playlist()` data
- Remove MATPU guard and switch data source to CF Workers before production release
- This is HIGH PRIORITY TODO — without it, production cannot launch

**Decision date:** 2026-04-07
