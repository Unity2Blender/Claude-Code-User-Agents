---
name: Backup-restore damage control — users + billing backfill
description: 2026-04-22 backup restore wiped recent users/billing rows; backfill flow now supports --since slice + billing INSERT
type: project
originSessionId: f0d434da-97f9-4d12-bc47-465747837ead
---
On 2026-04-22 a Supabase DB wipe + restore from a 7-hour-old backup wiped both `users` and `billing` rows (and cascade-trigger downstream rows: `billing_settings`, free-tier `subscription`, 6 module stubs) for hundreds of users who had signed in during the post-restore gap. Firebase Auth was unaffected.

**Why:** Backup restore is a recurring class of incident. The April 2026 backfill (`02a7830c`, 16,221 users) was a one-time reconciliation for a Cloud Function bug — different shape, but reuses the same scripts. The 2026-04-22 enhancements (commit `6a858667`) make the flow reusable for damage-control: `--since=<duration>` slice + automatic billing INSERT block.

**How to apply:** When a backup-restore or DB wipe incident occurs:

```bash
# Slice to the affected window (suffixes m|h|d)
./scripts/fetch-firebase-identified-users.sh --all --since=7h

# Generate users + billing SQL
./scripts/generate-backfill-sql.sh scripts/outputs/firebase_all_users.csv

# Eyeball, then apply
head -60 scripts/outputs/backfill_users.sql
psql "$DATABASE_URL" -f scripts/outputs/backfill_users.sql
```

**Key schema fact** (verified `000003_core_tables.sql`): `billing.user_id` is TEXT with `uq_billing_user_id` UNIQUE constraint, **NOT a FK to users.firebase_uid** (per the `COMMENT ON COLUMN` in 000003). So billing INSERT works standalone — but the script still emits users INSERT first for data integrity.

**2026-04-22 actual apply outcome:** Backfilled 517 users + 6,156 billing rows (cascade fired ~6,080×, creating ~30K downstream rows in subscriptions/billing_settings/warehouses/batch_config/batch_expiry_config). Final state: users 19,420; billing/subscriptions/billing_settings all 25,709 (1:1 invariant holds). users-without-billing eliminated (5,656 → 0). Backdating verified: backfilled rows show `created_at` matching Firebase CreatedAt (months-old), `updated_at` = apply time. All cascade rows used NOW() so trial windows start fresh.

**Diff-mode tool** (added 2026-04-22): `scripts/diff-firebase-supabase-and-backfill.py` does the verify-then-insert flow that the bash generator skips. Cross-references Supabase users + billing UID sets, emits SQL only for the actual diff, with backdated `created_at`. Runbook header documents the full workflow including chunk-and-MCP apply path when DATABASE_URL isn't available.

**Cascade triggers fire** on billing INSERT for: `billing_settings`, `subscriptions` (free-tier stub), and 6 module stubs. Verify with the cascade-sanity queries in the SQL footer (10-min `created_at` window).

**Why we backfill billing proactively** (vs letting `ensure_billing_exists` handle it on next sign-in): the user is offline post-restore. If we only backfill `users`, every RPC that reads `billing` fails until the user re-opens the app. Backfilling `billing` too restores the full row set immediately; cascade triggers fan out to all dependent module stubs.

Related migrations: 000003 (tables), 000010 (users.phone, users.auth_provider), 000046 (ensure_billing_exists), 000241 (embeds users upsert via JWT), 000277 (advisory lock).
