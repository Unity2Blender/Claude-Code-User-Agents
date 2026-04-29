---
name: Supabase migrations LATEST_MIGRATION sentinel location
description: The canonical migration sentinel file lives at `supabase/migrations/LATEST_MIGRATION`, not at the repo root. Always read this before proposing a new migration number.
type: reference
originSessionId: 2f07a826-81cd-48ae-ac41-c6677a99d6e5
---
The canonical sentinel file is `supabase/migrations/LATEST_MIGRATION`. Its content is the filename of the latest migration (e.g., `000423_admin_read_rpc_wrappers.sql`).

A `supabase/LATEST_MIGRATION` may exist at the parent directory but is **stale and not authoritative**.

Workflow before proposing a new migration:

1. Read `supabase/migrations/LATEST_MIGRATION` for the current head filename.
2. New migration number = `head + 1`, zero-padded to 6 digits.
3. Use `make migration-new NAME=...` to create the file (the Makefile updates `supabase/migrations/LATEST_MIGRATION` automatically). Never use `supabase migration new` (creates 14-digit timestamp, breaks the 6-digit convention).
4. The `validate-migrations` sentinel rejects non-6-digit prefixes and out-of-order files.

Cross-reference: `supabase/migrations/CLAUDE.md` (sentinel rules), `MIGRATE-012` (immutability rule, never edit shipped migrations), `WT-GOTCHA-005` (sentinel must exist before branching).
