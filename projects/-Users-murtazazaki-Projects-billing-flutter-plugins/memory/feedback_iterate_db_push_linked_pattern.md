---
name: Iterate `supabase db push --linked` to discover dependency cascades — `--dry-run` is insufficient
description: `supabase db push --linked --dry-run` only validates that migrations PARSE — it does NOT run them. Real dependency errors (DROP COLUMN with views/triggers/CHECKs, trigger-on-view restrictions, WHEN-clause column rules) only surface during apply. The reliable workflow is iterate-on-real-push, fix in place if not yet in schema_migrations, repeat. Document each iteration as audit trail in the migration's comment header.
type: feedback
originSessionId: d9f48ca5-88b8-449d-ae84-c7bfa9c170d6
---
## What `--dry-run --linked` does and does NOT catch

**Catches:**
- Filename pattern violations (e.g., `AGENTS.md` skipped).
- Migrations missing from `schema_migrations` (the "Would push" list).

**Does NOT catch:**
- SQL syntax errors inside migrations (parsed lazily by Postgres).
- Dependency errors (`DROP COLUMN ... because other objects depend on it`,
  SQLSTATE 2BP01).
- Trigger-on-view restrictions (`relation X is a view`, SQLSTATE 42809).
- WHEN-clause column-reference rules (`DELETE trigger's WHEN cannot
  reference NEW`, SQLSTATE 42P17; `column TG_OP does not exist`,
  SQLSTATE 42703).
- Schema-rename leftovers (a trigger that lived on a table now lives on
  the renamed table; references to the old name silently no-op).
- Cross-table CHECK or trigger parity invariants
  (`CTA Ownership Contract §4`, SQLSTATE 23514).
- Column-cascade audit trail (PostgreSQL CASCADE drops dependent objects
  silently; the migration log doesn't capture which CHECKs/triggers/views
  vanished).

## The reliable iteration loop

```
loop:
  1. supabase db push --linked --include-all
     (no --dry-run; we WANT real apply against real schema)
  2. If GREEN ─────────────────────────────────► done.
  3. If RED:
     a. Read SQLSTATE + error message (PostgreSQL is precise: it names
        the offending column / view / trigger / constraint by FQN).
     b. Inspect the failed migration. If it's NOT yet in
        schema_migrations (txn rolled back on apply failure), edit in
        place — that's the standard recovery pattern (NOT a violation
        of MIGRATE-012; the migration is "never deployed").
     c. If it IS in schema_migrations, write a compensating migration
        000NNN+1 that drops/recreates the broken object. Never edit an
        applied migration.
     d. Document the failure + fix in the migration's comment header
        (audit trail for future reads of the migration file).
     e. Re-run from step 1.
```

## Iteration history exemplar (2026-04-25 banner ↔ reel 1:1 refactor)

For the `tutorials_reels.video_url_mp4` drop in 000376, the loop ran 5
times before GREEN. Each iteration's failure surfaced a new dependency
that no amount of pre-flight grep would have caught comprehensively:

| Iter | SQLSTATE | Cause                                                  | Fix                                                  |
|------|----------|--------------------------------------------------------|------------------------------------------------------|
| 1    | 42703    | `is_sentinel` doesn't exist on tutorials_reels        | Removed AND filter                                  |
| 2    | 42809    | `CREATE TRIGGER ... ON public.tutorials` (a view)     | Target `tutorials_reels` instead                    |
| 3    | 42703    | `TG_OP` not allowed in trigger WHEN clause            | Pure column DISTINCT                                |
| 4    | 42P17    | DELETE trigger WHEN can't reference NEW               | Drop WHEN clause; filter inside function body      |
| 5    | 2BP01    | DROP COLUMN blocked by dependent view + trigger      | DROP VIEW + trigger first; recreate view at end    |
| 6    | GREEN    | applied                                                | —                                                    |

For the `tutorial_banners.cta_config_override` drop in 000377, 1 more
iteration:

| Iter | SQLSTATE | Cause                                                          | Fix                                                |
|------|----------|----------------------------------------------------------------|----------------------------------------------------|
| 7    | 2BP01    | `trg_normalize_cta_phone_banner_override` refs override col   | Added explicit DROP TRIGGER before column drop    |

For 000374 (sentinel CTA): 1 iteration:

| Iter | SQLSTATE | Cause                                                                      | Fix                                                          |
|------|----------|----------------------------------------------------------------------------|--------------------------------------------------------------|
| 0    | 23514    | CTA Ownership Contract §4 parity — changed cta_config.target_id without cta_feature_target_key | Convert A5 to verify-only no-op (target IS already correct) |

Each iteration cost ~30 seconds (one `db push --linked` round-trip). The
total was ~5 minutes of iterate-fix-iterate. NO amount of pre-migration
grep would have caught all 7+ dependency edges with the same precision —
PostgreSQL's catalog is the source of truth, and the apply-time error
messages are PRECISE (they name the dependent object by FQN).

## Pre-flight discovery (auxiliary, not a substitute)

Before iterating, do a best-effort grep:

```bash
# For DROP COLUMN <col> on table <T>:
grep -rn "<col>" supabase/migrations/*.sql | grep -v "$(this migration)"
# Look for: CHECK constraints, trigger fn bodies, trigger WHEN clauses,
# RPC RETURNS TABLE / SELECT, partial index WHERE, FK references,
# view column projections.

# For ALTER TABLE <T> on a renamed table:
grep -rn "RENAME TO\|ALTER TABLE.*RENAME" supabase/migrations/*.sql
# Find any prior renames; targets may need updating.

# For trigger-on-view:
docker exec -i supabase_db_<project> psql -U postgres -d postgres \
  -c "SELECT relkind FROM pg_class WHERE oid = 'public.<T>'::regclass;"
# 'r' = regular table; 'v' = view; 'm' = materialized view.
# Triggers can only be created on 'r'.
```

But pre-flight grep ONLY catches what's in the migrations folder. It misses:
- pg_proc functions defined inline by other migrations.
- Triggers that moved with prior renames.
- Views that other migrations CREATE OR REPLACE'd to add column references.
- Partial indexes whose WHERE clause silently uses the column.

The catalog is the source of truth. `pg_get_triggerdef`, `pg_get_viewdef`,
`pg_constraint`, `information_schema.triggers` queries against the live
DB are more reliable than grep — but even those miss future code that
hasn't been authored yet.

## Compensating-migration pattern when in-place isn't allowed

When the failed migration IS in schema_migrations (rare but possible —
e.g., the migration succeeded but exposed a logic bug downstream), the
fix is a NEW sequential migration that supersedes the broken one. Never
edit a migration whose version row is in `schema_migrations`.

Distinguish:
- **In-place edit OK**: migration's `version` is NOT in `schema_migrations`
  (apply failed; txn rolled back). The next `db push` retries from
  scratch with the edited content.
- **Compensating migration required**: migration's `version` IS in
  `schema_migrations`. Add 000NNN+1 with `CREATE OR REPLACE FUNCTION`
  / `DROP CONSTRAINT IF EXISTS` + `ADD CONSTRAINT` / similar.

## Workflow companion: the `--include-all` flag

`supabase db push --linked --include-all` retries ALL pending migrations
not in `schema_migrations` — including ones that previously failed. The
default mode (without `--include-all`) sometimes balks at retrying a
failed migration; `--include-all` is more forceful.

## Audit-trail discipline

Every iteration that fixed a SQLSTATE in a migration MUST update the
migration's comment header to capture:
1. The failed iteration's SQLSTATE + error class (`42703 column doesn't
   exist`, `42P17 DELETE trigger can't reference NEW`, etc.).
2. The fix applied.
3. Why the original migration was wrong (root cause).

This makes the migration self-documenting for future readers + for the
next agent that hits a similar dependency cascade.

## Companion lesson (also in `feedback_drop_column_cascade_audit.md`)

`SET LOCAL` is a no-op in Supabase CLI migrations (autocommit per
statement). `supabase db execute` doesn't exist. The audit script for
local dev should fall back to `docker exec ... psql` when psql is missing
on PATH.
