---
name: PG17 has no `CREATE ROLE IF NOT EXISTS` — use DO-block guard
description: PostgreSQL 17 CREATE ROLE has no IF NOT EXISTS clause. Wrap in a guarded DO block.
type: feedback
originSessionId: 83fb8343-bb17-4517-a0c9-061d55c22211
---
PostgreSQL 17's `CREATE ROLE` does NOT support `IF NOT EXISTS`. Using that syntax produces a parse error. Wrap with a guarded DO block.

**Why:** User caught this during v3 rebaseline plan review. `CREATE TABLE IF NOT EXISTS` exists; `CREATE ROLE IF NOT EXISTS` does not. Confusing because they look symmetric.

**How to apply:** Whenever a migration or baseline must idempotently ensure a role exists:
```sql
DO $$
BEGIN
  IF NOT EXISTS (SELECT 1 FROM pg_roles WHERE rolname = 'custom_role') THEN
    CREATE ROLE custom_role NOLOGIN;
  END IF;
END $$;
```
The same pattern applies to `CREATE POLICY` (also no `IF NOT EXISTS`): guard with `IF NOT EXISTS (SELECT 1 FROM pg_policies WHERE polname=... AND schemaname=...) THEN ... END IF;`.
