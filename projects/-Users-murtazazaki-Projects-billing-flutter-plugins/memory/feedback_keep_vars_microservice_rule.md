---
name: keep_vars microservice deployment rule
description: ALL Cloudflare Workers must use keep_vars=true to prevent wrangler deploy from wiping Dashboard-set vars. Applied across all microservices.
type: feedback
---

ALL Cloudflare Worker wrangler configs MUST include `keep_vars = true` (TOML) or `"keep_vars": true` (JSONC) at the root level.

**Why:** `wrangler deploy` default behavior (`keep_vars = false`) performs a destructive replace — deletes ALL remote vars, then sets only what's in config. This silently wiped Dashboard-set vars on every deploy, breaking production when local config had missing/incomplete values. User confirmed they had set vars via Cloudflare Dashboard that were being wiped.

**How to apply:**
- When creating new Cloudflare Workers, always add `keep_vars` to the wrangler config
- When auditing existing Workers, check for `keep_vars` presence
- This is a microservice-level standard — applies to payment-gateway (4 workers), bill-ocr, tutorials, supabase-proxy, and any future Workers
- Syntax: `keep_vars = true` (TOML) or `"keep_vars": true` (JSONC), always at root level (not per-environment)

**Workers requiring this (as of 2026-04-08):**
- `microservices/payment-gateway/packages/webhook-worker/wrangler.jsonc`
- `microservices/payment-gateway/packages/orders-worker/wrangler.jsonc`
- `microservices/payment-gateway/packages/catalog-worker/wrangler.jsonc`
- `microservices/payment-gateway/packages/gateway-router/wrangler.jsonc`
- `microservices/bill-ocr/wrangler.toml`
- `microservices/tutorials/wrangler.toml`
- `microservices/supabase-proxy/wrangler.toml`
