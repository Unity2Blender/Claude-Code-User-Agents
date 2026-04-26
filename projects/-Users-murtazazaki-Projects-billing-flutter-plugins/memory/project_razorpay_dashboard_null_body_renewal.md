---
name: Razorpay dashboard Request/Response=null on renewal webhooks is undocumented
description: Dashboard UI shows null body for payment.captured/order.paid on subscription renewals while showing full body for subscription.authenticated/cancelled/initial events. No RP docs, no community signal, no industry parallel. Worker logs are authoritative.
type: project
originSessionId: 9946a655-6d60-46e6-ae1e-61ad32ef5db3
---
**Fact (2026-04-15 observation + web research):** Razorpay's dashboard webhook-detail panel (`dashboard.razorpay.com/app/webhooks`) displays `Request: null` and `Response: null` for certain renewal-triggered events, while showing full bodies for other events on the same merchant account.

**Pattern (user-documented):**

| Event type | RP dashboard body |
|------------|-------------------|
| `subscription.authenticated` | ✅ full body |
| `subscription.cancelled` | ✅ full body |
| `payment.captured` on prepaid / initial subscription charge | ✅ full body |
| `payment.captured` on **renewals** | ❌ `null` |
| `order.paid` on **renewals** | ❌ `null` |
| `subscription.charged` on renewals | (likely `null` — to confirm) |

**Research results (targeted web-search-researcher, 2026-04-15):**
- **No Razorpay docs / changelog / FAQ** mentions per-event-type body display.
- **No community signal** on Reddit, Stack Overflow, GitHub, Svix, IndieHackers.
- **No comparable platform** (Stripe, Paddle, LemonSqueezy) does this.
- **No public API** returns historical delivered webhook bodies — dashboard is the only first-party surface.
- **No privacy/regulatory basis** — renewal bodies contain no PAN/CVV.
- **Most plausible hypothesis (speculation):** renewal/auto-charge path writes event rows via a different internal pipeline than user-facing payments; the dashboard's body column is populated only by the interactive path.

**Why:** Real incident 2026-04-15: renewal webhook failure investigation revealed the RP dashboard UI is unreliable for renewal events; only our Cloudflare Worker's HMAC-verified raw_payload is the authoritative record.

**How to apply:**
- Never rely on RP dashboard "Request" panel to diagnose renewal webhook payloads — check `webhook_events.raw_payload` in Supabase.
- When user screenshot shows `Request: null` in RP dashboard, this is EXPECTED for renewal events, not evidence of RP failing to send the body.
- If a future RP dashboard update fixes this, add a CLAUDE.md note and update this memory.
- Support ticket template is pre-staged in `.claude/plans/silly-watching-pearl.md` §12; file only if wedge doesn't resolve today's incident class (per D21).
