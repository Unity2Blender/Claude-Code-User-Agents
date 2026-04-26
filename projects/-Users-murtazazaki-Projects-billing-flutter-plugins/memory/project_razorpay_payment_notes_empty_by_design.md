---
name: Razorpay payment.notes = [] on renewals is documented behavior
description: payment.entity.notes is always empty on auto-charge subscription renewals per Razorpay docs — not a bug; anchor on subscription.entity.notes + subscription_id API lookup instead
type: project
originSessionId: 00d5ac66-32db-499b-b97c-b4ba118a188a
---
Razorpay's **official sample payload** for `subscription.charged` webhook events explicitly shows `payment.entity.notes = []` on the embedded payment entity. Only `subscription.entity.notes` preserves custom metadata across renewals.

**Why:** Razorpay auto-charges subscription renewals programmatically — there is no checkout opportunity to attach notes to the payment. Notes on the payment entity are a field reserved for notes set at payment-creation time by custom integrations.

**Concerning:** Razorpay's sample payloads for `subscription.paused` and `subscription.resumed` also show `notes: []` on the subscription entity — may be a docs-sample artifact OR may indicate notes get dropped on state transitions. Not confirmed without sandbox test or support ticket.

**Critical operational constraints:**
- Razorpay Dashboard webhook replay is a **manual support ticket** (not self-service)
- **15-day age limit** on replay — events older than 15 days cannot be replayed at all
- Razorpay **auto-disables webhooks after 24h** of persistent non-2xx responses (email alert sent to configured address)
- Ordering NOT guaranteed + at-least-once delivery → always dedupe via `x-razorpay-event-id`
- No per-endpoint rate limit documented; respond to HTTP 429 with exponential backoff + jitter

**Compare Stripe:** `invoice.subscription_details.metadata` is **immutable post-creation** — a snapshot frozen onto every renewal invoice. Razorpay offers no equivalent; must emulate via subscription-id API lookup.

**Why:** Originally appeared as "broken webhook" during v14/v15/v16 rollout; agent audit (2026-04-14) confirmed this is Razorpay's documented contract, not a regression. Saves future engineers hours of confused hunting.

**How to apply:**
- When debugging missing entitlement grants on subscription renewals, NEVER check `payment.notes.firebase_uid` first — it's guaranteed empty on auto-charges
- Always anchor on `subscription.entity.notes` (payload Layer 1) or `GET /v1/subscriptions/{id}` (API Layer 3)
- For `payment.captured` events with `payment.entity.subscription_id` set, route through subscription-id resolution
- Source: https://razorpay.com/docs/webhooks/payloads/subscriptions/ ; https://razorpay.com/docs/webhooks/faqs/ ; https://razorpay.com/docs/webhooks/best-practices/
