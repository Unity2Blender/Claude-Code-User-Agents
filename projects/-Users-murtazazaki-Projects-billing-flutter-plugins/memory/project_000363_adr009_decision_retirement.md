---
name: 000363 ADR-009 §Decision retirement (2026-04-23)
description: Two prod 42501 waves in 24h traced to 000360's anon-revoke; 000363 restored anon + authenticator EXECUTE on all 87 billing-gated SECDEF; ADR-009 §Decision retired for SECDEF RPCs.
type: project
originSessionId: e4a3da24-0e58-4bd3-8954-436f09de5b1a
---
## Arc (2026-04-22 → 2026-04-23)

| Mig | Role | Outcome |
|---|---|---|
| 000359 | REVOKE anon+PUBLIC on 2 balance RPCs + paired GRANT | Safe (template) |
| 000360 | Bulk REVOKE anon+PUBLIC on 87 SECDEF via ILIKE loop; **forgot GRANT block** | Root cause — wave 1: authenticated over-revoke |
| 000361 | Body-gate `IF NOT has_billing_access() THEN RETURN` on 4 balance RPCs | Cross-tenant safety — KEPT |
| 000362 | Narrow authenticated+service_role restore (compensator for 000360) | Closes wave 1 |
| 000363 | Broad anon + authenticator restore + explicit helper GRANTs | Closes wave 2 + retires ADR-009 §Decision |

## Root cause (wave 2)

000362 restored `authenticated` EXECUTE but kept anon revoked. Authenticated users whose JWT transiently lacks `role:"authenticated"` (Firebase custom claim missing on legacy cached tokens, JWT hydration race, refresh window) map to anon at PGRST's `db-anon-role` fallback. Anon cannot execute SECDEF funcs → 42501 on (a) `has_billing_access` via `parties_select` RLS policy USING clause, (b) `search_parties` direct RPC, (c) 85 other user-facing SECDEF RPCs. Prod evidence: 06:05 UTC (has_billing_access via RLS), 06:59 UTC (search_parties).

## Fix semantics (000363, committed `ee94910d`, deployed by operator 2026-04-23 ~06:50 UTC)

- **Part 1**: Explicit `GRANT EXECUTE ... TO anon, authenticated, authenticator, service_role` on the 3 RLS helpers (has_billing_access, user_billing_id, firebase_uid). Load-bearing for ~25+ tables' polroles='-' RLS policies.
- **Part 2**: Idempotent loop restores anon + authenticator on all 87 billing-gated SECDEF (same ILIKE filter 000360/000362 used).
- **Part 3**: 2 post-condition DO-blocks assert invariant (all 4 roles executable; 3 helpers anon-executable).
- **Part 4**: `NOTIFY pgrst, 'reload schema'`.
- **Why broad > narrow**: v3 audit surfaced `search_parties` 42501 within 10min of v3 narrow plan. Any JWT-role-claim-missing path touches ≥1 of 87; narrow leaves 86 bleeding. Body-gates + transport guard cover.

## Layered protection post-000363

1. **ADR-001 client-side transport guard** (`matpuSafeJwtProvider`) = sole MATPU boundary. Purely-anonymous Firebase users (no email + no phone) cannot present a JWT.
2. **000361 body-gates** (`IF NOT has_billing_access(...) THEN RETURN/RAISE`) = cross-tenant safety. Run regardless of role.
3. **000363 relaxation** = anon calls SECDEF → body-gate returns empty (firebase_uid()=NULL path) instead of hard 42501. Matches pre-000360 steady state.

## Verification evidence

- Live MCP on prod `qflwmecfufxrfddkykuk`: 87 total, 0 anon/authenticator/authenticated/service_role broken; 3 helpers anon=TRUE.
- Postgres log: last `permission denied for function has_billing_access` at timestamp 1776929315099000 (06:48 UTC); 000363 applied at 1776929458 (06:50 UTC); zero has_billing_access 42501 in the 20+ min post-deploy.
- Local: 32/32 fix-relevant pgTAP tests green (rls_helpers_anon_executable, payment_path, onboarding_rpc_access_guards, pgTAP setup).

## ADR-009 §Decision fate

RETIRED for SECDEF RPCs (header status flipped to `retired`; no in-place §Post-Incident Update #2 appended per feedback_retire_adrs_by_archival_not_amendment). Archival/trim to `docs/decisions/archive/` + rule codification (ACL-001, ACL-002, ACL-003, BHVR-024, BHVR-025, TEST-TIER-004, MIGRATE-018) + doctrine pgTAP tests (pgrst_pre_request_hook_acl, polroles_public_rls_helpers_acl, rpc_body_gate_returns_empty_for_anon) deferred to follow-up organization plan per feedback_p0_fix_minimum_viable_defer_doctrine.

## Track B (open follow-up, ≤7 days)

Client-side billing-gated RPC gateway provider. Wraps `postgrestProvider.rpc` with `authHydratedProvider.future` + `withAuthRetry` + error classifier. Refuses to present JWTs without `role:"authenticated"` claim at the client. Structural fix for the JWT-role-claim-missing vector that 000363 only tourniquets. Scoped in session-summary's "transport-guard skeleton; retire per-call wrapping" prompt.

## Permanent test gates

- `critical/rls_helpers_anon_executable_test.sql` (NEW, plan(4)) — anon EXECUTE on 3 helpers + body-gate behavioural assertion.
- `critical/payment_path_rpc_matpu_grants_test.sql` (REWRITTEN) — positive invariant (authenticated + service_role EXECUTE granted; PUBLIC never granted).
- `critical/rpc_execute_acl_anon_is_denied_test.sql` (DELETED — encoded retired rule).
- `critical/public_rpc_grants_allowlist_test.sql` (DELETED — encoded retired rule).
- `standard/onboarding_rpc_access_guards_test.sql` (header doctrine note added; assertions untouched — body-gate RAISES 42501 so throws_ok still valid).

## Files

- `supabase/migrations/000363_relax_anon_billing_gated_secdef.sql`
- `supabase/migrations/LATEST_MIGRATION` → 000363
- `docs/decisions/ADR-009-authenticated-role-execute-acl-matpu-client-boundary.md` (status: retired)
- Plan: `/Users/murtazazaki/.claude/plans/summary-of-the-session-s-bubbly-lantern.md` (rev v5)
- Commit: `ee94910d` on main
