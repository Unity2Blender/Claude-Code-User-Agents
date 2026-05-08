---
name: Migration scope gate before linked db push
description: Always inspect the pending migration set (not just the named target) before any supabase db push --linked. A blanket push applies ALL pending migrations including unrelated work in flight.
type: feedback
originSessionId: 2c5fcb68-16a2-4df2-b8c9-85d16216119a
---
P0 HARD — Before any `supabase db push --linked`, verify the EXACT pending set, not just the named target migration. `supabase db push --linked` applies every pending migration, in order, in the same connection.

**Why:** 2026-05-02 plan rejection. My plan said "push 000441" but `LATEST_MIGRATION` already pointed to `000442_adhoc_writeoff_accounting_and_banner_placement_class.sql`. A naive `supabase db push --linked` would have applied BOTH 000441 (search forward fix) AND 000442 (unrelated billing concern) in the same transaction batch. The user correctly flagged this as a scope hazard. Pushing two unrelated migrations in one operator gate creates two failure modes and ambiguous attribution if the smoke fails.

**How to apply:**
- Before any linked-mutating push step in a plan, the plan MUST include:
  1. `cat supabase/migrations/LATEST_MIGRATION` — name the head sentinel.
  2. `supabase migration list --linked` — enumerate every Local-not-Remote row.
  3. Compare the pending set vs the migration the plan is about to ship. If the set has unrelated migrations, STOP and surface them to the user; do not proceed with a blanket push.
  4. If unrelated work is pending and out of scope, the plan must explicitly choose: (a) defer the named migration until the unrelated work ships first, or (b) bundle them under one user gate with explicit per-migration smoke, or (c) ask the user.
- This applies even when MIGRATE-022 says the named migration is ledger-empty — that gate proves only the named migration is editable, not that the push is scoped.
- `supabase db push --linked --include <file>` does not exist; the push is always all-or-nothing over the pending set. The only way to scope a push is to commit and push migrations in sequence with separate operator gates per push.
