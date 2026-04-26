---
name: Razorpay Webhook Auto-Disable Gotcha
description: Razorpay auto-disables webhook endpoints after 24h of non-2xx — causes silent renewal failures; 2nd incident Apr 2026
type: project
---

Razorpay auto-disables webhook endpoints after 24h of continuous non-2xx (404 = 500, no differentiation). Exponential backoff retries, exact intervals undocumented. Re-enable is Dashboard-only (no standard merchant API). No self-service replay (contact support, <15 days, one at a time).

**Why:** If webhook-worker route is deregistered (gateway-router catch-all returns 404 for /webhooks/*), Razorpay accumulates failures and silently disables. Once disabled, ALL events stop — including subscription.charged renewals.

**How to apply:**
- webhook-worker MUST always return 200 after HMAC verification, even for unknown event types
- After deploying, ALWAYS verify webhook route exists: `bash scripts/verify-deploy.sh`
- NEVER deploy individual workers — ALWAYS `make deploy-all` (webhooks first, router last)
- If webhook auto-disabled, re-enable in Dashboard (Account & Settings → Webhooks)
- The v14 reconciliation cron (every 15 min) catches missed subscription renewals
- Recovery script: `npx tsx scripts/recover-entitlements.ts --dry-run --from ... --to ...`

**Incidents:**
- 2026-03-30: webhook-worker not deployed; gateway-router catch-all returned 404. Fixed by deploying.
- 2026-04-04-05: webhook-worker was deployed (v14), worked briefly, then route was deregistered after deploying another worker. ~20h outage, ~10-50 users lost entitlements. Fixed by redeploying + CI/CD pipeline.
- 2026-04-06: Hono `strict: true` (default) rejected `/webhooks/razorpay/` trailing slash → 404. Fixed by setting `strict: false` on all 4 workers. Test/sandbox events only (no user impact). Also fixed: orders + catalog workers had `"*"` for @payment-gateway/shared (should be `"file:../shared"`).
- 2026-04-07: Orphan `RAZORPAY_WEBHOOK_SECRET` (no suffix) on Cloudflare Dashboard → 401 HMAC failures.
- 2026-04-08: 503 SERVICE_CONFIG_ERROR — dual cause: (1) SUPABASE_SECRET_KEY was valid `sb_secret_*` format (~42 chars) but validation hardcoded `length < 100` rejecting it; (2) RAZORPAY_WEBHOOK_SECRET_LIVE was set to wrong 11-char value. Fixed by updating validation + setting correct 32-char secret. Real paying users affected.

**Root cause (Apr 4-5):** Per CF Workers docs, `wrangler deploy` only manages routes for its own worker. Route eviction happens if another worker's wrangler.jsonc temporarily contained `/webhooks/*` then was deployed without it.

**DLQ bug (fixed Apr 5):** Queue handler always called `handleQueue()`, never `handleDlq()`. Failed RC grants silently lost. Fixed: route based on `batch.queue === 'rc-retry-dlq'`.

**Deploy order (fixed Apr 5):** Reversed from router-first to specific-workers-first: webhooks → orders → catalog → router. Prevents catch-all from absorbing webhook traffic during deploy window.

**Razorpay IP whitelist:** `18.96.225.0/26` and `18.99.161.0/26`.

**Subscription notes behavior (confirmed):**
- `subscription.entity.notes` — HAS original notes (always, across all lifecycle events)
- `payment.entity.notes` — EMPTY on auto-generated renewal payments
