---
name: user_billing_id() STABLE caching gotcha in pgTAP tests
description: PG STABLE function caches session-wide when called from functions with SET search_path; causes wrong billing_id across JWT switches
type: project
originSessionId: c96fe297-452d-494c-9bd9-54d920a7b117
---
# STABLE function session-caching bug

`public.user_billing_id()` is declared `SECURITY DEFINER STABLE PARALLEL RESTRICTED SET search_path = pg_catalog, public, pg_temp` (migration 000003_core_tables.sql:73–96). Calling `firebase_uid()` internally and returning the resolved billing_id.

## Symptom

Inside pgTAP tests (and any long-running transaction that switches `request.jwt.claims` via `set_config(..., TRUE)` between statements), `user_billing_id()` returns the billing_id of the FIRST JWT it was called under — even after subsequent `set_config` calls change the JWT to a different user's `sub`.

Reproduced 2026-04-16 while writing `supabase/tests/database/critical/user_lightweight_summary_test.sql`:
- Test 4: JWT=user_A → RPC → firebase_uid()=user_A → user_billing_id()=A_billing
- Test 9: `set_config` → JWT=user_B → RPC → firebase_uid()=user_B but user_billing_id() STILL returns A_billing
- Trace via direct psql confirmed `firebase_uid()` correctly returns user_B but `user_billing_id()` is cached.

## Why: Why: `SET search_path` + STABLE + SECURITY DEFINER + no-args function triggers session-level memoization in PostgreSQL's function result caching. STABLE's contract is "same result within a single scan" but PG extends this to repeat-call cache when the function has no parameters AND its search_path is locked AND it's SECURITY DEFINER — likely because the planner treats the call as equivalent across statements.

## How to apply

When building new RPCs that need the caller's billing_id AND will be exercised in pgTAP tests with multiple JWT switches:

**DO NOT** call `public.user_billing_id()` inside plpgsql.

**DO** read JWT claims directly + inline the billing lookup:
```sql
v_claims_raw := current_setting('request.jwt.claims', TRUE);
IF v_claims_raw IS NULL OR v_claims_raw = '' THEN RETURN; END IF;
v_uid := NULLIF(TRIM(v_claims_raw::json->>'sub'), '');
IF v_uid IS NULL THEN RETURN; END IF;
SELECT id INTO v_billing_id FROM public.billing WHERE user_id = v_uid;
```

Canonical example: `supabase/migrations/000278_user_lightweight_summary.sql`.

Production doesn't exhibit this (Supabase connection pools don't switch JWTs mid-connection), but the pattern is still safer for future-proofing + test determinism.

## Where this matters

Not every RPC needs this — most RPCs are called once per HTTP request against a fresh JWT. The gotcha only matters when:
- The RPC is consumed by pgTAP tests that exercise cross-tenant or auth-transition scenarios.
- The RPC is re-entrant or called multiple times per transaction.

When in doubt, inline the JWT read. Cost: ~5 lines of plpgsql. Benefit: deterministic tests + robust to future PG planner changes.
