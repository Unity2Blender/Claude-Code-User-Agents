---
name: cta_feature_targets is SQL source of truth + parity invariant + audit both layers
description: Before assuming a CTA target is "fake" or unregistered, query public.cta_feature_targets — the SQL registry. Before mutating cta_config.target_id on tutorials_reels, atomically update cta_feature_target_key (parity trigger 000303 raises 23514 otherwise). Audit BOTH the SQL registry AND the Flutter FeatureTargetLabelRegistry — they can disagree.
type: feedback
originSessionId: d9f48ca5-88b8-449d-ae84-c7bfa9c170d6
---
## Three rules from the 2026-04-25 incident

### 1. `public.cta_feature_targets` is the SQL source of truth for CTA validity

Before declaring a target like `reels.browser` "broken" or "Worker-only
pseudo-target", query the SQL registry:

```sql
SELECT key, label_en, feature_module, nav_route, is_active
  FROM public.cta_feature_targets
 ORDER BY key;
```

Ground truth columns: `key` (PK), `nav_route`, `feature_module`, `is_active`.
Migration 000302 created the table; 000368 deleted `gst_calculator`. Phase 1
audits MUST include this query before any "target X doesn't exist" claim.

### 2. CTA Ownership Contract §4 — parity invariant

Trigger `trg_tutorials_cta_target_parity` (BEFORE INSERT OR UPDATE OF
`cta_config`, `cta_feature_target_key` ON `public.tutorials`, migration
000303:127-) raises SQLSTATE 23514 when:

```
cta_feature_target_key IS NOT NULL
AND cta_config->>'target_id' IS NOT NULL
AND cta_feature_target_key <> cta_config->>'target_id'
```

Net: any UPDATE that changes ONE of the two MUST also change the other in
the same statement. Atomic mutation pattern:

```sql
UPDATE public.tutorials_reels
   SET cta_feature_target_key = 'tutorials.videos_tab',
       cta_config = jsonb_set(cta_config, '{target_id}', '"tutorials.videos_tab"')
 WHERE id = 31;
```

Migrations that touch only one of the two will fail on apply with the
parity error. The `error message` itself names both columns and their
mismatched values — read it carefully.

### 3. Audit BOTH layers — SQL + Flutter — they can disagree

The SQL `cta_feature_targets` registry is independent from the Flutter
`FeatureTargetLabelRegistry`. A target key can be:

- **SQL-registered but not Flutter-dispatched**: the row exists in
  `cta_feature_targets`, but the Flutter dispatcher map (`_dispatch` in
  `cta_resolver.dart` + `FeatureTargetLabelRegistry`) lacks a handler.
  User-visible: tap fires, dispatcher returns `UnknownTarget`, no
  navigation. This is the `reels.browser` situation as of 2026-04-25.

- **Flutter-dispatched but not SQL-registered**: the dispatcher has a
  handler, but the SQL parity trigger rejects writes. Surfaces as
  SQLSTATE 23514 on content-ops attempts to seed the target.

Phase 1 audits MUST query both:

```bash
# SQL registry
docker exec -i supabase_db_<project> psql -U postgres -d postgres \
  -c "SELECT key, nav_route FROM public.cta_feature_targets;"

# Flutter dispatcher (grep for the target_id in the dispatch map)
grep -rn "FeatureTargetLabelRegistry\|'<target_id>'" \
  lib/billbook/tutorials/state/cta_resolver.dart \
  lib/billbook/tutorials/routing/
```

A "broken target" finding requires evidence from BOTH layers.

## How to apply (process)

For every migration that changes a `cta_feature_target_key` or a
`cta_config.target_id`:

1. Query `public.cta_feature_targets` to confirm the target IS / IS NOT
   registered.
2. Mutate BOTH columns atomically (single UPDATE statement) — never one
   without the other.
3. If introducing a new target, INSERT into `cta_feature_targets` FIRST
   (in the same migration or an earlier one), then UPDATE the consumer
   reels.
4. If the target is registered but the dispatcher doesn't dispatch, the
   fix is in Flutter (`cta_resolver.dart` dispatch table + the route
   handler at the named `nav_route`), NOT the SQL.

## Why (incident details)

A plan to retire the sentinel reel id=31's CTA target moved
`cta_config.target_id` from `'reels.browser'` to `'tutorials.videos_tab'`
without inserting `tutorials.videos_tab` into `cta_feature_targets` and
without updating `cta_feature_target_key`. Migration 000374 raised:

> tutorial id=31: cta_feature_target_key=reels.browser does not match
> cta_config.target_id=tutorials.videos_tab. CTA Ownership Contract §4
> requires parity. (SQLSTATE 23514)

Root cause: Phase 1 audit (delegated to an Explore agent) called
`reels.browser` a "Worker-only pseudo-target" based on it being absent
from the Flutter `FeatureTargetLabelRegistry` — without checking the SQL
`cta_feature_targets` table where it IS registered with
`nav_route='/reels'`. The migration was authored on the false premise
that the target needed replacing.

Fix: A5 retracted to a verify-only no-op. The sentinel reel's existing
parity (`cta_feature_target_key = cta_config.target_id = 'reels.browser'`)
is preserved. If the operator later wants a different target, the
migration sequence is INSERT-then-atomic-UPDATE per rule 2 above.
