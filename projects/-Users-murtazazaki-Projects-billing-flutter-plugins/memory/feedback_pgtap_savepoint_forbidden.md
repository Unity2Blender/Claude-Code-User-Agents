---
name: SAVEPOINT in pgTAP wipes the test counter
description: Never use SAVEPOINT + ROLLBACK TO SAVEPOINT inside a pgTAP file. Use SET LOCAL ROLE + RESET ROLE between blocks instead. SAVEPOINT wipes pgTAP's session-scoped test counter → finish() reports "No tests run!" even on passes.
type: feedback
originSessionId: 29573519-30ff-4263-9565-9f2fa2074e35
---
**Rule:** SAVEPOINT + ROLLBACK TO SAVEPOINT inside a pgTAP test file wipes the session-scoped test counter — `finish()` reports "No tests run!" even when every assertion would have passed. For role-switched pgTAP (testing RLS or SECDEF grants under `anon` vs `authenticated`), use `SET LOCAL ROLE <role>` + `set_config('request.jwt.claims', ...)` + `RESET ROLE` between blocks.

**Why:** Documented directly in `supabase/tests/database/critical/rpc_execute_acl_anon_is_denied_test.sql:25-33` after a 2026-04-22 incident. My v1 ACL plan (later audited and corrected) re-proposed SAVEPOINT inside pgTAP despite the file's own rule — caught by user audit before ship. pgTAP's `throws_ok`/`is_empty`/`lives_ok` already run their SQL inside PL/pgSQL `BEGIN/EXCEPTION/END` sub-transactions; 42501 errors ARE caught without poisoning the outer transaction. SAVEPOINT adds nothing and breaks the counter.

**How to apply:**
- Role-switched pgTAP pattern:
  ```sql
  SET LOCAL ROLE anon;
  SELECT throws_ok($$SELECT ...$$, '42501', NULL, 'message');
  RESET ROLE;

  SELECT set_config('request.jwt.claims', '{"sub":"...","role":"authenticated"}'::text, TRUE);
  SET LOCAL ROLE authenticated;
  SELECT is_empty($$SELECT ...$$, 'message');
  RESET ROLE;
  ```
- Outer `BEGIN; SELECT plan(N); ... SELECT * FROM finish(); ROLLBACK;` wraps the whole file — no SAVEPOINTs needed.
- If a review suggests SAVEPOINTs for "test isolation," decline and cite `rpc_execute_acl_anon_is_denied_test.sql:25-33`.
- Applies ONLY to pgTAP files. Application SQL code can use SAVEPOINTs freely.
