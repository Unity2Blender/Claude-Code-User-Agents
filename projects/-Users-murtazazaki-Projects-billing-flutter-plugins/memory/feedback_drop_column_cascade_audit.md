---
name: DROP COLUMN requires explicit cascade enumeration
description: Before any ALTER TABLE DROP COLUMN, grep migrations for all dependents (CHECK constraints, trigger functions, RPCs) and DROP each explicitly — PostgreSQL silently CASCADEs but the audit trail vanishes
type: feedback
originSessionId: d9f48ca5-88b8-449d-ae84-c7bfa9c170d6
---
When a migration drops a column, the dependents that PostgreSQL will silently
cascade-drop (or worse, leave broken) include:

1. **CHECK constraints** — any constraint whose expression references the
   column. Implicit CASCADE drops them silently — the audit trail is lost.
2. **Trigger functions** — function bodies are stored as text and
   late-bound. PostgreSQL allows DROP COLUMN even when functions reference
   the column; the function fails on next call (silent runtime error).
3. **Trigger WHEN clauses** — if the WHEN expression references the
   column, the DROP COLUMN raises 2BP01 dependency error UNLESS the
   trigger is dropped/recreated first.
4. **RPC bodies (RETURNS TABLE columns + SELECT)** — same late-bind
   semantics as trigger functions.
5. **Indexes** — partial indexes whose `WHERE` clause references the
   column are silently dropped.
6. **Foreign keys** — implicit cascade.
7. **Views / materialized views** — explicit dependency error unless the
   view is dropped/recreated.

**Why** (2026-04-25 incident):

A plan to drop `tutorials_reels.video_url_mp4` (Migration A3.6 / 000376)
did not enumerate the dependents. The migration would have:

- Implicitly cascade-dropped `chk_tutorials_production_requires_video`
  (000295) — losing the audit trail entirely.
- Left `fn_purge_tutorial_cache_on_update()` (000297, hardened 000301)
  in a broken state — the trigger fires on every tutorial mutation and
  would have silently failed at runtime once the column was gone, with
  the failure surfacing as "trigger function does not return EXPECTED
  type" or similar opaque error.
- Left `get_tutorial_by_id` v1 (000255) selecting a missing column —
  fails on next call (acceptable since superseded, but discoverable
  only via runtime monitoring).

A parallel plan to drop `tutorial_banners.cta_config_override` (Migration
A4 / 000377) only listed 1 of 11 dependent CHECK constraints. PostgreSQL
would have silently dropped the remaining 10 — losing the audit trail
that `chk_banner_override_contract_version`, `..._target_id_format`,
`..._navigate_has_target`, `..._paywall_has_gate`, `..._open_link_target`,
`..._whatsapp_phone`, `..._call_phone`, etc. ever existed.

**How to apply**:

For every `ALTER TABLE foo DROP COLUMN x` in a planned migration:

1. **Grep the migrations folder** for the column name:
   ```bash
   grep -rn "<column_name>" supabase/migrations/*.sql | grep -v "000NNN_drop_X"
   ```
2. **Categorize each hit**:
   - CHECK constraint → list with explicit `DROP CONSTRAINT IF EXISTS <name>;`
   - Trigger function body → list, `CREATE OR REPLACE` to remove the
     reference BEFORE the DROP COLUMN.
   - Trigger WHEN clause → DROP TRIGGER, recreate without the column.
   - RPC body → either CREATE OR REPLACE without the column, or accept
     the next-call failure (and document it in the migration comment).
   - Partial index WHERE clause → list, drop explicitly OR ensure the
     index is no longer needed.
3. **Order the migration**:
   - Drop trigger / CHECKs / dependent functions FIRST.
   - Recreate trigger functions / RPCs WITHOUT the column.
   - DROP COLUMN last.
4. **Embed a DO $verify$** block at the end asserting the post-drop
   shape matches expectations.

**Companion lesson — `SET LOCAL` does not work in Supabase CLI migrations**:

The Supabase CLI's `pgconn.Batch` transport (pgx PostgreSQL driver) sends
each statement in a migration file via the simple-query protocol with
autocommit semantics — no implicit BEGIN/COMMIT around the migration file
as a whole. `SET LOCAL lock_timeout = ...` and `SET LOCAL statement_timeout
= ...` emit `WARNING: SET LOCAL can only be used in transaction blocks`
and have no effect. Either:

- Drop the SET LOCAL statements from migration files. The implicit
  per-statement atomicity provided by the CLI is the actual contract.
- OR explicit BEGIN/COMMIT is forbidden by the project rule (per
  supabase/CLAUDE.md "Transaction Atomicity (CLI-Handled)").

Net: don't bother with SET LOCAL in migrations. Set timeouts at the
session level via psql env vars or accept the database defaults.

**Companion lesson — `supabase db execute` does not exist as a CLI command**:

The Supabase CLI's `db` subcommand has `start`, `reset`, `push`, `pull`,
`diff`, `dump`, `lint`, `test` — but NOT `execute`. To run ad-hoc SQL
against a linked DB, use `psql` against the connection string (Dashboard
→ Database → Connection string → URI), or write a temporary migration
that runs the query via `DO $$ ... $$` and `RAISE NOTICE` for output.

**Companion lesson — migration recovery pattern (failed apply)**:

When `supabase db push --linked` fails mid-batch on migration N, the
CLI rolls back the txn — schema_migrations does NOT record N as applied.
N is "never deployed". In-place edits to N are the standard recovery
pattern, distinct from MIGRATE-012's prohibition on editing applied
migrations. The next `db push --linked` retries N from scratch.

The strict interpretation of MIGRATE-012 (write a new migration N+1 that
supersedes N) does not work here because `db push` will retry N first
and fail again until N itself is fixed.
