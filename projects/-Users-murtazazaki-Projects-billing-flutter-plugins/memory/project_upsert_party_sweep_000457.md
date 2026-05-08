# Project: upsert_party SWEEP 000457

Date: 2026-05-06
Status: Active

`upsert_party` is SWEEP class as of
`supabase/migrations/000457_upsert_party_restore_to_sweep.sql`.

Why:

- Production 2026-05-06 showed April-4 clients failing party creation with
  SQLSTATE 42501 after anon EXECUTE was removed.
- Missing-role Firebase JWTs execute as `anon`, but still carry `sub`.
- `upsert_party` authorizes through `has_billing_access(p_billing_id)`, which
  reads `firebase_uid()` from the JWT `sub`.

Artifacts:

- ADR: `docs/decisions/ADR-024-upsert-party-restore-to-sweep.md`
- Spec: `docs/specs/upsert_party_restore_to_sweep_forward_fix.md`
- Contract: `supabase/tests/database/critical/upsert_party_sweep_contract_test.sql`

Current CLOSED remainder from the 000456 set:

- `save_payment_with_allocations(jsonb,jsonb,boolean,bigint)`
- `save_payment_with_allocations_v2(jsonb,jsonb,jsonb,text,boolean,boolean,bigint)`

Follow-up only with evidence/idempotency proof: payment v1/v2 SWEEP audit and
`commit_item_master_patch` classification audit.
