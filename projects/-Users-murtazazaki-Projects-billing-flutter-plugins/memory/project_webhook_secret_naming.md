---
name: Webhook Secret Naming Conventions
description: Two naming conventions for Razorpay webhook secrets — Convention A (Workers, canonical) vs Convention B (Cloud Run legacy, dead code)
type: project
---

Two naming conventions coexist for Razorpay webhook secrets:

**Convention A (canonical — used by webhook-worker):**
- `RAZORPAY_WEBHOOK_SECRET_LIVE` / `RAZORPAY_WEBHOOK_SECRET_TEST`
- Used in: `wrangler.jsonc`, `WebhookEnv` type, `index.ts` handler, `secrets-mapping.json`, `Makefile`

**Convention B (legacy — dead code from Cloud Run):**
- `RAZORPAY_LIVE_WEBHOOK_SECRET` / `RAZORPAY_TEST_WEBHOOK_SECRET`
- Used in: `shared/src/config/credentials.ts` (`loadCredentialsFromEnv`), root `.dev.vars.example`
- The `webhookSecret` field in `credentials.ts` is populated but never consumed for HMAC verification

**Why:** Naming changed during GCP Cloud Run → Cloudflare Workers migration. Convention B code was not removed.

**How to apply:** Always use Convention A when referring to webhook secrets. Convention B code in `credentials.ts` is dead — targeted for removal.

**2026-04-07 incident:** Orphaned `RAZORPAY_WEBHOOK_SECRET` (no suffix) on Cloudflare Dashboard caused confusion. Real 401 errors on `order.paid` and `payment.captured` events.
