---
name: CF wrong Supabase URL — auth sync gap
description: authOnUserCreate Cloud Function deployed with tunnel URL instead of production Supabase; caused 2160:432 billing/users gap
type: project
originSessionId: 08f9484c-efb3-465a-8845-e84f7d7535e2
---
Cloud Function `authOnUserCreate` was deployed with `SUPABASE_URL=https://supabase-dev.gstapps.com` (test tunnel) instead of production URL (`https://qflwmecfufxrfddkykuk.supabase.co`). CF was successfully writing users rows to the wrong database.

**Why:** `functions/.env` contained tunnel credentials. No project-specific `.env.calc-e6vzmi` file existed. `firebase deploy` bundles `functions/.env` as runtime env.

**How to apply:** Always verify `functions/.env` has production Supabase credentials before deploying Cloud Functions. Consider adding a deploy-time validation script. The `defineString` defaults were removed to fail deployment if vars are missing.

**Additional fixes applied (2026-04-11 plan):**
- Embedded users upsert inside `ensure_billing_exists()` as server-side fallback
- Flipped CF error handling: unknown errors now throw (retryable) instead of silently returning
- Downgraded Node.js 22 → 20 (Gen1 compatibility)
- `backfillMissingUsers` to reconcile ~1700 orphaned users
