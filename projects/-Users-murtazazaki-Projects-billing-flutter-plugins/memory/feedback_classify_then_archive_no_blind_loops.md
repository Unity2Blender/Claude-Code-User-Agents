---
name: Classify-then-archive migrations, never blind loops
description: When archiving migrations during a rebaseline, classify each file first against linked ledger + DDL coverage. Blind `mv 000001..N-1` is unsafe.
type: feedback
originSessionId: 83fb8343-bb17-4517-a0c9-061d55c22211
---
When squashing/rebaselining migrations, never `git mv` every file blindly. Classify each pre-baseline migration into one of four classes first.

**Why:** A file may be local-only (not yet pushed to linked), or contain DML/data ops that the prod-derived baseline does not capture (e.g. seeding). Blindly archiving loses those. User's exact correction: *"Step 4's archive loop is still too broad."*

**How to apply:** For each migration with version < NEXT_NUM:
1. **Class A — archive (in baseline):** applied in linked ledger AND its DDL/data effect is captured by the prod-derived baseline → `git mv` to `migrations_archive_v3/`.
2. **Class B — preserve after baseline:** local-only and still needed → renumber as `${NEXT_NUM+k}_<name>.sql`, keep in `supabase/migrations/`.
3. **Class C — archive with rationale:** obsolete/local-only and not needed → `git mv` to archive with a documented reason in `migrations_archive_v3/README.md`.
4. **ABORT — unknown classification:** halt and ask the user before continuing.

Read the linked ledger snapshot (`.claude/.rebaseline-v3-evidence/linked_ledger_pre.txt`) as the source of truth for "applied in linked." Inspect non-DDL ops (seeds, data fixes) manually before deciding. This step is INTERACTIVE; never auto-pick on ambiguity.
