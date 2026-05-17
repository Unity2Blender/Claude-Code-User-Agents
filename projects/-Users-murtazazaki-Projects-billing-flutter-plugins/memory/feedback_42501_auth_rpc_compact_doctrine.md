---
name: 42501 Auth/RPC compact doctrine (umbrella) — claim merge, ACL classification, withAuthRetry semantics
description: Three-sub-section reference extracted from the prior single 2121-char MEMORY.md kernel bullet (L91) by V5.2-FW SW2 compaction. Kernel now carries 3 narrower P0 HARD entries pointing here for the verbose body. Covers Firebase claim-merge + JWT role routing; ACL classification (CLOSED for financial RPCs vs SWEEP for body-guarded user-facing); withAuthRetry semantics + log redaction.
type: feedback
originSessionId: V5.2-FW-SW2-2026-05-17
---

# 42501 Auth/RPC Compact Doctrine

Extracted from MEMORY.md L91 (the single 2,121-char P0 HARD bullet) by V5.2-FW SW2 compaction. The active kernel now carries 3 narrower P0 HARD entries — one per sub-section below — pointing here for the verbose body.

Cross-links to existing dedicated files:
- [[feedback_42501_retry_policy_call_site_opt_in]] — call-site opt-in via `retryablePermissionDeniedFunctions` parameter
- [[feedback_sweep_guard_p0001_mapper_parity]] — SWEEP guard + P0001 mapper parity invariants
- [[feedback_postgrest_sweep_rpc_classification]] — PostgREST SWEEP RPC classification rules
- ADR-023, ADR-024 — architectural decisions on auth/RPC posture

## Auth claim merge + JWT role routing

Firebase Admin `setCustomUserClaims` replaces the WHOLE custom-claims object — fetch existing `customClaims` and merge before setting `role: "authenticated"` to avoid losing other claims. PostgREST `user_name: authenticator` is only the connection role; the effective request role comes from JWT role switching. Missing/no-role Firebase JWTs route to Supabase `anon`. ACL hardening alone does NOT recover stale clients — only claim repair plus token refresh / re-auth / token expiry does. Missing-role repair UX is explicit repair-then-retry: user taps Retry → callable repairs own role claim → client force-refreshes Firebase/Supabase auth → original RPC reissues.

Firebase Auth V1 `onCreate` triggers do NOT repair claims on sign-in. Per-sign-in blocking repair requires Identity Platform; existing-user repair runs through admin/support tools.

## ACL classification + financial-RPC exception

SECDEF RPC ACLs use:

- **CLOSED** for financial / write / state-mutation RPCs: revoke `PUBLIC` / `anon` / `authenticator`, grant `authenticated` and `service_role` only. Helper-based identity extraction through `firebase_uid()`, `has_billing_access`, `user_billing_id`, or JWT-claim reads is a SWEEP candidate ONLY for non-financial user-facing RPCs. Financial / write / state-mutation RPCs stay CLOSED unless a human approves an exception with idempotency, duplicate-submit, and lock/allocation proof.
- **SWEEP** only for explicitly body-guarded user-facing RPCs. Body guard pattern: early null-auth guard + canonical bare `RAISE EXCEPTION 'Not authenticated';` / P0001 + anon grant so clean auth UX survives role-claim races.

Never grant client RPC execution to `authenticator`.

Payment / Razorpay / subscription write RPCs must NOT enter 42501 retry without per-RPC idempotency proof (see Razorpay renewal classifier + abandonment doctrine).

## withAuthRetry semantics + log redaction

`withAuthRetry` already force-refreshes through existing `SupabaseRestClient.refreshAuth(forceRefresh: true)` / client recreation paths — do NOT add a third raw `getIdToken(true)` strategy inside it. Callers MUST read `postgrestProvider` INSIDE the retry closure (capturing the client outside breaks Strategy 2 client recreation; the force-recreate disposes the old PostgrestClient). 42501 retry policy is call-site opt-in via `retryablePermissionDeniedFunctions` parameter — no global blanket-retry of "permission denied for function". See [[feedback_42501_retry_policy_call_site_opt_in]] for the narrower call-site rules.

**PGRST\* values** are PostgREST `error.code` values; **42501 / P0001** are PostgreSQL SQLSTATEs.

**Log redaction**: client `[AUTH_RETRY]` logs must NOT contain raw UID / sub / token / JWT material. Server Functions logs MAY include explicit UID fields (the server is the auth boundary; the client is not).
