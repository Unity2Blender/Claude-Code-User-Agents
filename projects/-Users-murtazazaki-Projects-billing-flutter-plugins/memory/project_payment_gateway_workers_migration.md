---
name: Payment Gateway Workers Migration
description: Cloud Run → Cloudflare Workers migration completed for payment-gateway (3 workers at pay.gstapps.com)
type: project
---

Payment gateway migrated from Cloud Run (Express) to 3 Cloudflare Workers (Hono) at `pay.gstapps.com` as of 2026-03-29.

**Why:** Lower latency, no cold starts, zero GCP costs for payment processing, edge caching for catalog.

**How to apply:**
- Production URL: `PAYMENT_GATEWAY_PROD_URL=https://pay.gstapps.com` (flutter.env)
- Subscription paths: canonical `/api/subs/*`, legacy `/api/subscriptions/*` aliases exist temporarily
- Workers: `catalog-worker`, `orders-worker`, `webhook-worker` in `packages/`
- Legacy Cloud Run Express code in `src/` is deprecated but kept for reference
- Webhooks (Razorpay + RevenueCat) already live on Workers
- Deploy: `make deploy-all` from `microservices/payment-gateway/`
- Tests: `make test-cf` runs Vitest with Miniflare (21 tests across 3 workers)
- Vitest configs use `include: ['src/**/*.test.ts']` (relative to package dir, NOT monorepo root)
