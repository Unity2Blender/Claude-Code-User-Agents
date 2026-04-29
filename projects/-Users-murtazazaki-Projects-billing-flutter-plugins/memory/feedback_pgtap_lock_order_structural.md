---
name: pgTAP can't prove concurrent lock order — use structural pg_proc.prosrc assertion
description: When verifying deterministic FOR UPDATE order in functions, assert the source pattern, not concurrent behavior
type: feedback
originSessionId: b5603ad8-f84c-4a63-bf3f-ede3326973a9
---
pgTAP runs in a single transaction with `BEGIN; ... ROLLBACK;` and can't reliably prove concurrent lock acquisition order. Two transactions touching invoices [A,B] vs [B,A] would deadlock in production but pgTAP doesn't have multi-session primitives to exercise that path.

**Replacement pattern:** Use a structural assertion against `pg_proc.prosrc`:

```sql
SELECT ok(
    (SELECT prosrc FROM pg_proc
     WHERE proname = 'save_payment_with_allocations'
       AND pronamespace = 'public'::regnamespace
     LIMIT 1
    ) ~ 'ORDER BY \(value->>''invoice_id''\)::BIGINT',
    'structural: function body contains ORDER BY invoice_id deterministic loop sort'
);
```

The regex `~` is case-sensitive by default — match the exact casing in the function body.

**Why:** Reference 2026-04-28 audit feedback ("pgTAP won't reliably prove concurrent lock order"); used in `52-save_payment_clamp_phase0_test.sql` for both the new-allocation loop sort AND the edit-mode pre-lock pattern.

**How to apply:** When writing pgTAP for any function that uses `FOR UPDATE` or advisory locks, prefer structural source-pattern assertions over behavioral lock-order tests. Pair with rule citations from `audit-sql-concurrency` skill (LOCK-*, DL-*) for review-time clarity.
