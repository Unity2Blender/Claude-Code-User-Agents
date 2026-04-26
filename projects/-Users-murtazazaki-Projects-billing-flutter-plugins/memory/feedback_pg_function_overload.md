---
name: PostgreSQL function overload prevention
description: CREATE OR REPLACE with different param count creates a new overload — always DROP old signature first
type: feedback
---

When adding parameters to a PostgreSQL function, always DROP the old signature first — `CREATE OR REPLACE FUNCTION` with a different parameter count creates a new overload, not a replacement. This causes 42725 ambiguity errors when callers use named arguments.

**Why:** Migration 20260320100000 used `CREATE OR REPLACE` to add `p_reorder_level` to `set_opening_stock`, creating a 6-arg overload alongside the M11 5-arg version. When `upsert_item_with_stock` called with 3 named args, PostgreSQL couldn't disambiguate.

**How to apply:** Always use `DROP FUNCTION IF EXISTS ... (old_signature); CREATE OR REPLACE FUNCTION ... (new_signature);` in the same migration (atomic via CLI implicit transaction). Also update `schemas/functions/*.sql` to match, or `squash.sh --generate-only` will re-inject the old version into M11.
