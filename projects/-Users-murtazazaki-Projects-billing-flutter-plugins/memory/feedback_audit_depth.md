---
name: Audit Depth — Verify Runtime Wiring
description: Always verify that planned changes are actually reachable from the host app's call site, not just the library interface
type: feedback
---

When auditing a library module (e.g., reels in billing_flutter_plugins), always verify that the HOST APP actually passes the callbacks/params the plan depends on. Don't just audit the library interface — check the call site.

**Why:** In the Tutorial Engine v5 audit (2026-04-07), the plan proposed CTA auto-resume, analytics wiring, and retry enhancements — but the GST app's `videos_home_view.dart` doesn't pass `ctaCallbacks`, `onSupport`, or `onAnalytics` into `ReelsView`. The entire hardening plan was unreachable from the host app. Also missed: the seen provider marks ALL videos seen on category open (contradicting the per-video decision), and the migration bulk-enables all categorized rows as reels-eligible (step-overlay rows bleed into playlists).

**How to apply:** For any library module audit:
1. Read the host app's call site (not just the library widget constructor)
2. Check what params are actually passed vs. declared
3. Verify seed data matches the intended behavior (don't trust migration comments)
4. Check provider hydration timing (async hydrate after returning defaults = nondeterministic first frame)
