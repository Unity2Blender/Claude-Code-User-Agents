---
name: Purchase mode price sync bug takes priority over prewarm plan
description: User rejected ExitPlanMode for prewarm plan because purchase mode price sync bug is more urgent; address user's current bug report before closing prior plans
type: feedback
originSessionId: c96fe297-452d-494c-9bd9-54d920a7b117
---
When the user reports a new runtime bug during plan review, address the new bug FIRST before trying to close the prior plan. The user rejected ExitPlanMode because they wanted the purchase mode price sync issue investigated immediately.

**Why:** The user is actively testing the app and finding bugs in real-time. The prewarm plan can wait — the actively-broken purchase flow is more urgent.

**How to apply:** When user pastes runtime logs + screenshots showing a new bug while in plan mode for a different feature, pivot to investigating the new bug. Don't try to close the prior plan first.
