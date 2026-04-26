---
name: RC entitlement nomenclature
description: RevenueCat entitlement IDs use hyphens (mobile-gold, mobile-silver, web-basic, web-gold), NOT underscores. Platform-tier format.
type: project
---

RevenueCat entitlement identifiers use **hyphens**, NOT underscores:
- `mobile-silver` (GST Calculator silver)
- `mobile-gold` (GST Calculator gold)
- `web-basic` (Web billing basic)
- `web-gold` (Web billing gold)

**Why:** The code at `revenuecat_notifier.dart:351-352` had `mobile_gold`/`mobile_silver` (underscores) which NEVER matched the actual RC entitlement keys. This was the root cause of RC entitlements being overshadowed — every user fell through to the Supabase RPC fallback.

**How to apply:** Any code checking RC entitlement keys must use hyphens. The naming convention is `{platform}-{tier}` (e.g., `mobile-gold`, `web-basic`).

**Directive:** RC is the ultimate source of truth for entitlements. If RC says paid, do NOT also check Supabase. Supabase is the fallback only when RC has NO entitlements.
