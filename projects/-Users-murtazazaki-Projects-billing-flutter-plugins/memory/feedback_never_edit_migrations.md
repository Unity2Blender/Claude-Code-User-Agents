---
name: Never Edit Deployed Migrations
description: All migration files are locked after creation — always create new sequential compensating migrations, never in-place edits
type: feedback
originSessionId: 52a8b03f-a491-467e-b2e2-42e0a968ed1c
---
NEVER edit an existing migration file. Always create a new sequential migration file (compensating migration).

**Why:** This is a production-linked project. `supabase db push` tracks timestamps only — modified content is NEVER re-applied. Editing a deployed migration creates silent schema drift between local (`db reset` replays all) and production (`db push` skips already-applied). The user explicitly stated: "all the migration files, they are supposedly locked" and "just bother to create a new migration file."

**How to apply:**
- When a migration needs fixing: create `000(N+1)_fix_description.sql` with DROP + CREATE
- NEVER use `supabase migration repair` unless absolutely necessary
- Reference existing migrations by reading them, but treat content as immutable
- Even on feature branches before merge: prefer new files for traceability (user chose this for 000239 over editing undeployed 000238)
- Added as MIGRATE-012 (Critical) and MIGRATE-013 (High) in design-postgres-tables skill rules
