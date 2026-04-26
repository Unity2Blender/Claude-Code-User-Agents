---
name: camelCase vs snake_case Razorpay Notes Mismatch (v16)
description: Flutter RazorpayNotesPayload.toJson() produces camelCase keys but webhook handler expected snake_case — caused 3 failed entitlement grants on 2026-04-08
type: project
---

**Incident 2026-04-08:** 3 real paying users got no entitlement after payment. Root cause: `extractRazorpayNotes()` in `webhook-worker/src/index.ts` only checked `firebase_uid` (snake_case), but `payment.entity.notes` contained `firebaseUid` (camelCase) from Flutter's `RazorpayNotesPayload.toJson()` passed via Razorpay SDK checkout options.

**Why:** Razorpay SDK checkout `notes` become **payment-level notes** on `payment.entity`, overriding order-level notes. The orders-worker sets snake_case keys, but the SDK checkout passes the Flutter client's camelCase keys. Subscription events are unaffected because `subscription.entity.notes` use server-set snake_case keys.

**How to apply:**
- `extractRazorpayNotes` now accepts BOTH formats: `notes?.firebase_uid ?? notes?.firebaseUid` and `notes?.package_id ?? notes?.planId`
- One-time orders now have full 3-layer UID fallback (payload → DB → API), matching subscriptions
- When adding new note fields in the Flutter client, ensure the webhook handler checks both camelCase and snake_case variants
- The `razorpay_orders` table may not exist when webhook arrives (race with verify route) — API fallback handles this

**Recovery:** Reset `attempt_count=0` on webhook_events IDs 372, 379, 383. After deploying fix, `/internal/replay-failed` or cron auto-recovers.
