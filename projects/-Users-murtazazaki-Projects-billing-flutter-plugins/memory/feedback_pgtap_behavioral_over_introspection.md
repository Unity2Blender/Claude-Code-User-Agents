---
name: pgTAP behavioral assertions outrank introspection
description: Behavioral pgTAP tests are primary correctness gates; pg_proc.prosrc ILIKE checks and RAISE NOTICE log assertions are best-effort guards only.
type: feedback
originSessionId: 0c3b1341-9b54-4e2b-950e-08f723a37edd
---
In pgTAP, prefer behavioral tests over function-source introspection or log-output assertions when proving correctness. Two specific rules:

1. **`pg_proc.prosrc ILIKE '%pattern%'` is a secondary guard, not a correctness gate.** It only proves the pattern appears in source — it does not prove the path executes correctly. Use it as a cheap sentinel that a code path is wired (e.g. `prosrc ILIKE '%PT409%'`) but never as the load-bearing assertion. Pair every prosrc check with a behavioral test that exercises the actual behavior end-to-end.

2. **`RAISE NOTICE` content cannot be reliably asserted in pgTAP.** Single-session pgTAP can sometimes capture NOTICE via session-level `client_min_messages`, but this does NOT prove the message reaches Supabase log explorer in production. Verify NOTICE/log emission manually after local smoke or post-deploy in production via the log explorer / `pg_stat_statements`. Keep pgTAP focused on what it can prove deterministically: SQLSTATE raised, DETAIL content, return values, side-effects on rows.

**Why:** Plan reviews repeatedly flag prosrc ILIKE assertions and NOTICE-content assertions as brittle. They drift silently when source comments change or when log levels differ between local and prod. Behavioral assertions (call the function, observe rows/errors/return values) survive refactors that move pattern strings into helper functions or change wording.

**How to apply:**
- pgTAP file authoring: write behavioral assertions FIRST (call the function, assert rows/SQLSTATE/DETAIL). Add prosrc checks ONLY when concurrency or lock-order is genuinely unprovable in single-session pgTAP — and keep them labeled as "structural guard" not "correctness gate".
- Plan documents: when a step says "assert X via prosrc ILIKE", flag it for a behavioral counterpart.
- For `RAISE NOTICE`-style observability, defer verification to a manual smoke step in the verification plan (post-`supabase migration up` or post-`db push --linked`), not to a pgTAP assertion.
- Related: `feedback_pgtap_lock_order_structural.md` — single-session pgTAP can't prove concurrent lock order; that's the one case prosrc-as-fallback is justified.
