---
name: 000360 over-revoke P0 incident (2026-04-22 → 2026-04-23)
description: 000360 bulk anon REVOKE stripped authenticated on 72 billing-gated SECDEF functions. Fixed by 000362 (narrow compensating GRANT) and 3915883d (client cold-start race). ADR-009 §Decision preserved; bulk-ILIKE migrations retired.
type: project
originSessionId: 29573519-30ff-4263-9565-9f2fa2074e35
---
**What happened:** 2026-04-22 — migration 000360 ran a dynamic `pg_proc` LOOP with `ILIKE '%has_billing_access%' AND has_function_privilege('anon', …, 'EXECUTE') = TRUE`, REVOKEd anon + PUBLIC EXECUTE on ~72 billing-gated SECDEF functions, but OMITTED the `GRANT EXECUTE TO authenticated, service_role` re-affirm block that sibling migrations 000333 (lines 91-118) and 000359 (lines 83-105) both ship. Authenticated users were riding on the implicit PUBLIC grant on legacy functions; `REVOKE FROM PUBLIC` stripped it. Production 2026-04-22 19:22-19:25 UTC: authenticated paying users hit `42501 permission denied for function` on `has_billing_access`, `should_show_inventory_wizard`, `get_party_ledger`, and ~69 more.

**What was fixed:**
- 2026-04-23 migration `000362_restore_authenticated_on_billing_gated_secdef.sql` (commit 16edc5c7) — narrow compensating GRANT on the 72 broken functions using filter `ILIKE '%has_billing_access%' AND NOT has_function_privilege('authenticated', oid, 'EXECUTE')`. Idempotent. Includes MIGRATE-017 post-condition DO-block.
- Moved `advisory/public_rpc_grants_allowlist_test.sql` → `critical/`, added `authenticated=TRUE` bidirectional invariant (plan(3)→plan(4)) — the missing half that let 000360 ship undetected for 24h.
- Extended `critical/rpc_execute_acl_anon_is_denied_test.sql` plan(9)→plan(15) with role-switched per-RPC blocks for `has_billing_access` + `should_show_inventory_wizard`.
- 2026-04-23 commit `3915883d` — client cold-start race hotfix. Post-000362 prod logs still showed 42501 at 04:36 + 05:07 UTC; MCP canary confirmed 85/85 auth=TRUE / 0 leaked anon. Root cause: `inventory_wizard_ticker.dart` + `party_dashboard_provider.dart` fired `.rpc()` / `.from()` before `authHydratedProvider` resolved → PostgREST role-switched to anon → ADR-009 correctly denied. Fix: add `await ref.watch(authHydratedProvider.future)` + `withAuthRetry` wrapping on both call sites; extend `_isJwtExpired` in `postgrest_retry.dart:379-388` to match `42501 + "permission denied for function"` for one force-refresh retry.

**Doctrine:**
- ADR-009 §Decision PRESERVED (anon revoked on billing-gated SECDEF).
- ADR-009 §Open Findings #1 resolution WITHDRAWN — bulk-ILIKE migrations RETIRED as a pattern.
- ADR-001 client-transport guard remains the MATPU boundary, not superseded.
- New rules codified (narrow, not blanket): FUNC-078 (paired REVOKE+GRANT on SECDEF public), MIGRATE-017 (banner claims as executable post-conditions), BHVR-022 (evidence-gated ACL hardening), BHVR-023 (role-switched pgTAP mandatory on SECDEF), TEST-TIER-003 (ACL meta-tests in critical tier).
- Deferred: CHAIN-ACL-001 (Flutter provider ACL sentinels), BHVR-WITHAUTHRETRY-001 (full `.rpc(` sweep), ORCH-001 (7-day 42501 bucketing pipeline), TRANSPORT-REGRESSION (integration test sentinel).

**Key files to look at if this recurs:**
- `supabase/migrations/000362_restore_authenticated_on_billing_gated_secdef.sql` — the compensating template
- `supabase/tests/database/critical/public_rpc_grants_allowlist_test.sql` — bidirectional meta-test (the lint gate)
- `docs/decisions/ADR-009-authenticated-role-execute-acl-matpu-client-boundary.md` §Post-Incident Update (2026-04-23)
- Plan: `/Users/murtazazaki/.claude/plans/event-message-permission-denied-bubbly-tide.md` (rev v2)

**Lesson:** Predictive bulk hardening (pg_proc introspection, ILIKE filters) is higher-risk than targeted hardening. Always include a bidirectional meta-test in the SAME PR — single-sided tests miss over-revoke. Encode banner-comment claims as executable post-conditions.
