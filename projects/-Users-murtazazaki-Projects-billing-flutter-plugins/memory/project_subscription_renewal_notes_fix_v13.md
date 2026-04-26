---
name: Subscription Renewal Notes Extraction Fix (v13)
description: P0 fix — webhook-worker extracted notes from payment.entity (empty on auto-renewals) instead of subscription.entity, causing MISSING_UID and lost entitlements
type: project
---

**Fixed 2026-04-03** — Commit `9eb2ddb2`, deployed as webhook-worker version `954df065`.

## Root Cause
`webhook-worker/src/index.ts` extracted notes from `payment.entity ?? subscription.entity`. On `subscription.charged` renewals, Razorpay's auto-generated payment has empty `notes`, but `subscription.entity.notes` carries `firebase_uid` + `package_id`. Code picked payment first → missed subscription notes → `firebaseUid = null` → `grantAndTrackRcStatus()` threw `MISSING_UID` → user paid but lost RC entitlement.

**Why:** The bug existed in 3 locations: main handler, replay endpoint, cron reconciliation — creating an infinite failure loop.

**How to apply:** When working on webhook notes extraction, always use `extractRazorpayNotes()` helper which is subscription-first. DB fallback enrichment from `razorpay_subscriptions` is belt-and-suspenders for missing notes.

## Key Razorpay Behavior (from research)
- `subscription.entity.notes` persists throughout lifecycle (all webhook events)
- `payment.entity.notes` on auto-generated renewals is `{}` (empty) — NOT copied from subscription
- No `subscription.renewed` event exists — renewals fire `subscription.charged`
- `paid_count` distinguishes first charge (1) from renewals (2+)
- Notes max: 15 key-value pairs, 256 chars each
- `PATCH /v1/subscriptions/:id` can update notes on existing subscriptions
