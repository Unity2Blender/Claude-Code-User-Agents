---
name: GST sentinel probe — client-side filter truncation bug
description: PostgREST .limit(N) + ORDER BY id ASC + client-side filter silently drops matching rows when ≥N non-matching rows precede them by sort key
type: project
originSessionId: e2f8e44b-274b-414f-a32e-6a5f6b0e99b4
---
**Incident:** 2026-04-22. `/v1/admin/health-data-sentinels` returned `healthy:false, actual:0` while `mcp_supabase.execute_sql` confirmed the canonical sentinel row (`tutorial_banners.id=59, placement_id='gst', is_sentinel=true, is_active=true, retired_at=null`) was present.

**Root cause — file `microservices/tutorials/src/routes/health.ts:425-444` (pre-fix):**

```ts
await supabase
  .from('tutorial_banners')
  .select('id, tutorial_id, status, is_sentinel, is_active, retired_at')
  .eq('placement_id', 'gst')
  .order('id', { ascending: true })
  .limit(5);
// then: rows.filter(r => r.is_sentinel && r.is_active && r.retired_at === null)
```

When ≥5 non-sentinel `placement_id='gst'` rows existed with `id<59`, `.limit(5)` returned those 5 and the sentinel (id=59) was truncated BEFORE the client-side filter. Result: `actual=0`, probe falsely reported unhealthy, `verify-invariants.sh` would have blocked the deploy even though prod was in the intended state.

**Fix — commit `ca60cf6e` (main, 2026-04-22):** push predicates server-side before `.limit()`. Drop the now-redundant client filter.

```ts
.eq('placement_id', 'gst')
.eq('is_sentinel', true)
.eq('is_active', true)
.is('retired_at', null)
.order('id', { ascending: true })
.limit(5);
```

Regression locked at `microservices/tutorials/tests/contract/health-data-sentinels.contract.test.ts` — the `REGRESSION:` test seeds 5 non-sentinel gst rows + the sentinel at id=59 and asserts `actual=1`. With the pre-fix code, that test returns `actual=0`.

**Pattern to watch for elsewhere:** any `.from(t).select().eq(col, v).order().limit(N)` followed by a `.filter()` is a truncation trap. If the filter condition is a column value (not a derived computation), push it into the query via `.eq`/`.is`/`.gt` before `.limit`. Production-relevant only when the base table has enough rows that `.limit(N)` can plausibly truncate matches.

**Pattern is NOT a bug when:** the filter condition is derived from joined data the PostgREST client can't express, or when `.limit(N)` is a hard upper bound for a UI paginator where client-side filtering is the intended UX.
