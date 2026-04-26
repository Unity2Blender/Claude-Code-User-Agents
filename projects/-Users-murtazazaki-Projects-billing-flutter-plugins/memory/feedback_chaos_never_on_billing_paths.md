---
name: Chaos NEVER on billing/payment/webhook paths
description: Hard rule -- do not apply chaos, progressive delivery, synthetic load, traffic shadow, or canary to Razorpay, subscription, GST-ledger, invoice-finalize, webhook-delivery, or any pg_net call path
type: feedback
originSessionId: 36859de4-f174-4b1e-a38b-23ec596500f8
---
**Rule:** NEVER apply chaos engineering, progressive delivery, synthetic load testing, traffic shadowing, or canary rollout to any billing, payment, subscription, GST-ledger, invoice-finalize, webhook-delivery, or `pg_net` call path. This includes:

- Razorpay webhook handlers (`functions/src/supabase-webhooks/`, any `subscription.*` / `order.paid` / `payment.captured` endpoint)
- Billing triggers (`functions/src/billing-triggers/`)
- Firebase -> Supabase auth-sync triggers (`firebase-auth-supa-db-triggers/`)
- SQL RPCs: `apply_payment_atomic`, `create_and_finalize_invoice`, `invoice_finalize`, `ensure_billing_exists`, `save_invoice_atomic`, any writer of `ledger_entries` / `subscriptions` / `payments` / `gst_transactions`
- Every `pg_net.http_post` / `pg_net.http_get` call site
- Codemagic release pipelines for GST Calculator (Play Store tag pushes)

**Why:** The 2026-04-08 lost-entitlements incident -- 3 paying users lost subscriptions because a camelCase/snake_case notes mismatch in a Razorpay webhook handler silently dropped entitlement grants -- is the canonical evidence. Chaos philosophy presumes the right to re-run. Payments have no re-run button. Progressive canary on a webhook endpoint would scale the loss with the canary percentage. Regulated GST invoice finalization can cause statutory breaches on duplicate issue races.

**How to apply:**

- Before any CHAOS-*, CANARY-*, DOGFOOD-*, or `--write-pattern=` chaos suggestion, check the path against the protected list in `.claude/skills/devops-architect/resources/reliability_frontier.md` (section 3).
- If the path is protected: refuse the suggestion and propose a deterministic test-env dry-run or staging replay instead.
- Pre-launch gated features are a different case -- see `feedback_prelaunch_direct_prod_testing.md`. The exception is narrow: build-gated internal test path hitting prod endpoint before Play Store launch. It is NOT permission to canary post-launch billing paths.
- When the enforcement surface is a load test, require the tool to fail setup/teardown if it detects a protected path (the `k6_load_profile.js` template has this built in -- aborts in `setup()` if `TARGET` contains `/webhook`, `/billing`, `/razorpay`, etc.).
- The k6 template tag `scope=non-billing` is not decorative -- treat its presence as a contract and its absence as a block.
