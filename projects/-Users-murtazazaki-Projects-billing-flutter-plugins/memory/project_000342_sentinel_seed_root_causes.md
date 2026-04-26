---
name: 000342 sentinel seed — 3 stacked root causes
description: Why make deploy-prod blocked on 000342 on 2026-04-21 and how 000350 compensates. Reference for future seed-migration authoring.
type: project
originSessionId: 149be99b-9154-46c2-8f4c-87bb5791f778
---
000342_gst_sentinel_seed.sql failed `expected exactly one active sentinel for placement=gst, got 0` on production 2026-04-21. After `supabase migration repair 000342 --status reverted --linked` the retry failed identically. Reproduced locally via `supabase migration up`. Three stacked bugs in 000342:

**Why:** Three bugs hid behind each other. DO NOTHING swallowed UNIQUE violations silently, so the INSERT producing 0 rows looked like success until verify checked. FK violation would have been the visible symptom IF the INSERT ever ran. Trigger GENERATED-column bug never surfaced because test 8 in critical/gst_sentinel_invariant_test.sql never ran end-to-end (migration never applied).

1. **Partial UNIQUE collision on tutorials_reels**
   - `uq_tutorials_surface_sort_active` — UNIQUE (surface_id, sort_order) WHERE is_active=TRUE. 000342 inserts `(reels_invoicing, 0, TRUE)` but row id=17 already occupies the slot.
   - `uq_tutorials_category_display_order` — UNIQUE (category, display_order) WHERE display_order IS NOT NULL. 000342 inserts `(invoicing, 1)` but row id=1 already occupies.
   - `ON CONFLICT DO NOTHING` WITHOUT target catches both silently.
   - **Fix in 000350**: sort_order=9999 (out-of-band), display_order=NULL (excluded by partial index predicate), explicit `ON CONFLICT (surface_id, step_id) DO NOTHING`.

2. **FK gap — `tutorials_cta_feature_target_key_fkey`** requires `cta_feature_targets.key = 'reels.browser'`. Never seeded when reels module landed.
   - **Fix in 000350**: `INSERT INTO cta_feature_targets (key='reels.browser', label_en='Explore reels', feature_module='tutorials', nav_route='/reels', ...) ON CONFLICT (key) DO NOTHING` BEFORE the tutorials_reels insert.

3. **GENERATED-column semantics bug in trigger** — `trg_guard_sentinel_banner` checks `NEW.is_active` in a BEFORE UPDATE trigger. Postgres docs §5.3: STORED generated columns are computed AFTER BEFORE triggers. `NEW.is_active` therefore always equals OLD value inside the trigger — the guard is a no-op. Caught by test 8 in critical/gst_sentinel_invariant_test.sql on first real run.
   - **Fix in 000350**: CREATE OR REPLACE FUNCTION checks `NEW.status IS DISTINCT FROM 'published'` instead.

**How to apply:**
- When authoring a seed that INSERTs into a table with partial UNIQUE indexes, enumerate EVERY partial UNIQUE before choosing values. Prefer out-of-band values (9999) or NULL (when the predicate filters `IS NOT NULL`).
- Always use explicit ON CONFLICT target so collisions on OTHER constraints RAISE instead of silently skipping.
- For CREATE TRIGGER BEFORE UPDATE on tables with STORED GENERATED columns, check the RAW source column (`NEW.status`) — never the generated one (`NEW.is_active`).
- When a migration fails on prod AND retry fails identically, the bug is in the migration itself — reproduce locally via `supabase migration up` (after `migration repair --status reverted --local` if needed).
- For recovery, `migration repair --status applied --linked` skips a broken file; a new compensating migration must bootstrap the DDL that never ran.

**Deploy sequence** (2026-04-21 unblock, plan `microservices-tutorials-make-deploy-pro-silly-bachman.md`):
```bash
supabase migration repair 000342 --status applied --linked
supabase db push --linked --dry-run   # expect 000343..000350 pending
supabase db push --linked
```
