---
name: GST Result Page action zone history
description: Action buttons (copy/history/share) were below the breakdown card in v2 FlutterFlow era — user moved them to AppBar and is now restoring original position
type: project
---

The GST Result Page action buttons (copy, history, WhatsApp share) were originally positioned below the GstBreakdownCard in the v2 FlutterFlow-era app. The user (Murtaza) was the one who moved them to the AppBar. The current redesign is a **restoration** of the original position — not a muscle memory disruption. This invalidates any A/B testing concerns about the swap.

**Why:** User considers the AppBar placement was a mistake — the body placement provides better thumb accessibility for 200K+ daily users.

**How to apply:** Do not flag muscle memory risk when evaluating layout changes that restore previous positions. The user has explicit history context that overrides brainstorming audit concerns.
