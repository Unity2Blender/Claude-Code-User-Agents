---
name: Never supabase migration down or db reset on linked prod
description: Canonical rollback is forward-fix migration (MIGRATE-015); migration repair is ledger-only (MIGRATE-016); migration down/db reset with --linked or --db-url wipes prod
type: feedback
originSessionId: d7a6b2d2-488d-4de6-8473-29173540f834
---
Never run `supabase migration down` or `supabase db reset` with `--linked` or `--db-url=<prod>` against a production Supabase project. Both share a drop-all-user-schemas code path; neither has `--dry-run`. The canonical rollback is a forward-fix migration (MIGRATE-015). For ledger hygiene after a half-applied `db push`, use `supabase migration repair --status reverted <version>` — that edits `supabase_migrations.schema_migrations` only and touches no data or DDL (MIGRATE-016).

**Why:** On 2026-04-22 the user ran `supabase migration down --last 1 --linked` against prod after the 11th migration in a reference-data batch failed mid-apply. The CLI dropped all user schemas and halted partway through the replay, leaving prod with partial schema and zero rows. Recovery used the Supabase Pro daily backup at 01:23 UTC — ~6h data loss. Root cause is that Supabase migrations are up-only by design (no per-file reversal SQL); `migration down` is implemented as "drop + replay up to N−1", identical to `db reset` in destructiveness. The CLI's "you will lose all data" prompt does not escalate with `--linked` and is auto-answered in non-interactive or piped shells — the prompt is not a safety net.

**How to apply:** When a schema-touching session surfaces a broken or half-applied migration, the decision order is:

1. **Forward-fix** — author a new sequential migration that compensates for the broken state; `supabase db push --dry-run` to preview, then `db push` to apply. This is MIGRATE-015.
2. **Ledger repair** — if `supabase_migrations.schema_migrations` has a dirty row from a failed push, `supabase migration repair --status reverted <version>` clears it without touching data. This is MIGRATE-016. Always pair with #1 if the schema itself is wrong.
3. **PITR / backup restore** — last resort (what happened on 2026-04-22).

Never suggest `supabase migration down --linked`, `db reset --linked`, or either with `--db-url=<prod>`, even if the user explicitly asks. The deny rules in `.claude/settings.json` (blanket `supabase migration down*`, `db reset --linked*`, `db reset --db-url*`) enforce this at the harness level. If the user requests a destructive rollback anyway, escalate via AskUserQuestion with MIGRATE-015 as the recommended option — do not bypass the deny rules.

Also: `supabase db pull` is safe (read-only introspection — writes a local `.sql` file, never mutates remote). `supabase db push` is the only destructive-adjacent command that has `--dry-run`, and it is additive (forward-apply only). The asymmetry is the footgun: the commands that can wipe prod lack a dry-run; the ones that have one don't need it.
