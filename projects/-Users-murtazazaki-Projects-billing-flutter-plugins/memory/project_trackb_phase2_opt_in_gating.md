---
name: Track B Phase 2 opt-in gating (matpuSafeJwtProviderWithRoleGate)
description: Role-gated JWT providers shipped in commit 36371bda are NOT wired in main.dart — activation is gated on confirming legacy-token prevalence via prod analytics before flipping the default.
type: project
originSessionId: e4a3da24-0e58-4bd3-8954-436f09de5b1a
---
## What shipped (commit `36371bda`, 2026-04-23)

Two new static helpers on `SupabaseRestClient`:

- `matpuSafeJwtProviderWithRoleGate(currentUser)`
- `matpuSafeForceRefreshJwtProviderWithRoleGate(currentUser)`

Both refuse to present JWTs that lack `role:"authenticated"` claim. Fail-open on `getIdTokenResult` errors (offline / network). Case-insensitive match. 14 critical sentinel tests green.

## Why NOT wired into main.dart

`lib/main.dart:104` and `apps/gst_calculator/lib/main.dart` continue to use `matpuSafeJwtProvider` (the non-gated variant). The swap to `*WithRoleGate` is deferred as **Track B Phase 2** — not part of the 000363 post-incident batch.

**Gating rationale:** the role-gated provider is a structural MATPU fix. Flipping it as the default has a surprise sign-out risk for users whose Firebase `authBeforeUserCreated` trigger never fired server-side (legacy signup path, trigger-disabled window, etc.). Those users have a valid session but a tokenbody missing the `role` custom claim — under the gated provider, they appear signed-out until Firebase re-issues a token that picks up the server-side claim.

Until we have prod analytics on how many users are in that state, a default-on flip could surface as "thousands of users lost their session overnight" — exactly the kind of regression we avoided with 000363's broad scope.

## Phase 2 activation checklist (when the time comes)

1. **Measure legacy-token prevalence.** Instrument the role-gated provider in debug build only to count:
   - Tokens with `role:"authenticated"` → the happy path.
   - Tokens with role absent / mismatched → the refused path.
   - Send counts (not tokens, not uids) via analytics.
2. **Backfill stale users.** If the refused-path count is non-trivial, run a one-shot backfill (Cloud Function iterates Firebase users, reissues claims) BEFORE the default flip.
3. **Wire the swap.** Change the two `main.dart` call sites:
   ```dart
   jwtProvider: SupabaseRestClient.matpuSafeJwtProviderWithRoleGate(...),
   forceRefreshJwtProvider: SupabaseRestClient.matpuSafeForceRefreshJwtProviderWithRoleGate(...),
   ```
4. **Extend matpu_postgrest_guard_test + billing_context_anonymous_guard_test** to assert the role-gated path.
5. **Watch Logflare for 2 hours.** Confirm 42501 rate stays at zero + `{sql_state_code:"42501"}` doesn't spike on RLS helpers.
6. **Retire the non-gated providers** (keep for test compat, delete later).

## Why not ship Phase 2 now

Operator doctrine (`feedback_p0_fix_minimum_viable_defer_doctrine`): the P0 fire is out; doctrine organization is tourniquet-adjacent but structural activation deserves its own interview + canary window. 000363 restored the empty-state / body-gate path for legacy-token users so the fallback no longer red-banners — Phase 2 can wait for the analytics signal.

## Related

- Commit `36371bda` (the provider + tests)
- Commit `ee94910d` (000363 P0 fix)
- `project_000363_adr009_decision_retirement.md`
- `feedback_p0_fix_minimum_viable_defer_doctrine.md`
- ADR-001 (transport guard remains sole MATPU boundary)
