---
name: OCR match_field override CHECK drift (2026-04-12)
description: Schema-code drift where Worker wrote 'override' but SQL CHECK rejected it; `/violates.*constraint/i` regex in isPermanentError amplified the damage; now gated by 3-way contract (TS Zod + SQL CHECK + pgTAP critical contract test)
type: project
originSessionId: 2dae6666-59af-48a1-bf35-0f98decfbfae
---
# OCR `match_field` Override CHECK Drift (2026-04-12)

## Incident
Production `refresh_ocr_candidates` RPC rejected `match_field='override'` with SQLSTATE 23514 (check_violation) on `chk_ocr_scan_party_candidates_match_field`. Same bug on sibling item-side CHECK. Every user who had a correction-memory match from a prior scan silently burned a Gemini credit and saw their scan marked permanently failed.

## Root Cause (three-layer drift)
- TS types `microservices/bill-ocr/src/types/scan.ts:99-113` declared `'override'` in union
- Worker runtime `microservices/bill-ocr/src/services/entityMatcher.ts:175,331` wrote `'override'` at 0.97 confidence in Phase 0 correction-memory injection
- PostgreSQL CHECK `supabase/migrations/000048_ocr_scans.sql:243,279` allowed only `('name','gstin','pan','phone','composite')` for parties and `('name','hsn','composite')` for items
- Amplifier: `scanProcessor.ts:461` regex `/violates.*constraint/i` classified the error PERMANENT → `review_status='failed'` + queue-ack (no retry) + Gemini credit burned

## Fix (Phases A-F shipped 2026-04-12)
- Migrations `000243` (party) + `000244` (item) widened CHECKs with NOT VALID + VALIDATE + preflight DO block + NOTIFY pgrst + ROLLBACK FORBIDDEN header
- `TRANSIENT_SQLSTATES` in `scanProcessor.ts` gained `23514` + `23505`; the `/violates.*constraint/i` regex removed from `PERMANENT_MSG_PATTERNS`
- `scanProcessor.ts` refresh_ocr_candidates + complete_ocr_processing error paths converted from `throw new Error(...)` to `throw new SupabaseRpcError(msg, sqlstate)` so SQLSTATE classification fires correctly; structured `logger.error('supabase_rpc_error', {...})` emitted for CF Analytics pressure-gauge
- Single-source taxonomy at `microservices/bill-ocr/src/types/matchFieldTaxonomy.ts` (const + Zod enum + assert guards)
- pgTAP contract test in `critical/` tier: `supabase/tests/database/critical/ocr_match_field_contract_test.sql` asserts `pg_get_constraintdef()` set-equality with TS taxonomy
- Recovery script: `scripts/recovery/2026-04-12-ocr-override-backfill.ts` (dry-run default, 10/min throttle for 1hr, then unthrottled; halts at 10% failure rate)

## Contract Source of Truth
- SQL CHECK (runtime authority)
- TS `matchFieldTaxonomy.ts` const + Zod enum (Worker authority)
- pgTAP critical contract (CI gate — blocks push if drift)

## Companion Sweep Decision (Q4)
User selected "Sweep all ~25 CHECK allowlists in same PR" — Phase E0 enumeration pending. When implementing new values: update TS, write NOT VALID+VALIDATE migration, update contract test.

## Why transient for 23514
Not because 23514 is always retryable — rather: schema migrations can temporarily violate until deployed; CF Queue has 3-retry + DLQ → dead_lettered; user can still retry via existing UI (`POST /scans/:id/retry`). Net effect: schema-code drift surfaces as transient → DLQ telemetry → engineers see it, users don't burn credits.

## Memory Pointer
See also: `feedback_never_edit_migrations.md` (MIGRATE-012/013), `project_billing_column_naming.md` (sibling schema-code contract bug on `billing.user_id`).

## Key Files (current as of 2026-04-12)
- Taxonomy: `microservices/bill-ocr/src/types/matchFieldTaxonomy.ts`
- Writers: `microservices/bill-ocr/src/services/entityMatcher.ts:175,331`
- Error classifier: `microservices/bill-ocr/src/services/scanProcessor.ts:TRANSIENT_SQLSTATES`
- Migrations: `supabase/migrations/000243_*` and `000244_*`
- pgTAP contract: `supabase/tests/database/critical/ocr_match_field_contract_test.sql`
- pgTAP override unit test: `supabase/tests/database/standard/103-ocr_candidate_refresh_rpc_test.sql` (tests 13+14)
- Jest: `microservices/bill-ocr/tests/unit/matchFieldTaxonomy.test.ts`, `scanProcessor.test.ts` (isPermanentError 23514+23505 transient cases), `entityMatcher.test.ts` (Phase 0 override injection for party + item)
- Recovery: `scripts/recovery/2026-04-12-ocr-override-backfill.ts`
- Rule: `CHECK-ALLOWLIST-001` in `supabase/migrations/CLAUDE.md`
