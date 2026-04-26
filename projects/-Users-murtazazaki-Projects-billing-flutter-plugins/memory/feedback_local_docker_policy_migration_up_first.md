---
name: Local Docker Policy — migration up first, db reset for bootstrap only
description: Default daily local-Docker maintenance is `supabase migration up`, not `supabase db reset`. db reset is reserved for bootstrap/recovery.
type: feedback
originSessionId: 83fb8343-bb17-4517-a0c9-061d55c22211
---
Default local-Docker maintenance command is `supabase migration up`, NOT `supabase db reset`.

**Why:** `db reset` replays every migration from zero. It carries the liability of every past file — when one old migration breaks (e.g. `000012_functions_views.sql` failure that caused the v3 rebaseline), `db reset` becomes painful. `migration up` only applies pending forward migrations and is the migration system's intended steady-state command. The user's exact words: *"Supabase DB reset has as a liability of maintaining these past migration files... we can only keep looking forward."*

**How to apply:**
- After any new migration: `supabase migration up && ./supabase/scripts/seed-sync.sh && make test-db-critical`.
- Reserve `supabase db reset` for: one-time rebaseline water-test (proving fresh bootstrap works), fresh developer-machine validation, unrecoverable local drift after diagnosis, release/freeze confidence checks.
- If `migration up` fails locally: inspect ledger and failing SQL → decide whether DB can be repaired safely (forward-fix migration locally) → use `db reset` only if drift is unrecoverable or local data is disposable.
- This rule lives as `LDP-001` in `.claude/skills/design-postgres-tables/data/rules/worktree.csv` (or new `local_docker_policy.csv`).
