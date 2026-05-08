# Feedback: SWEEP Guard P0001 Mapper Parity

Date: 2026-05-06
Status: Active

Canonical SWEEP body guard shape is:

```sql
IF public.firebase_uid() IS NULL THEN
  RAISE EXCEPTION 'Not authenticated';
END IF;
```

The bare raise defaults to SQLSTATE P0001 and matches the
`user_facing_rpcs_anon_body_guard_test.sql` drift regex.

When a user-facing RPC changes from planner-stage 42501 to a body-stage P0001
guard, audit both client layers:

- `postgrest_retry.dart::classifyAuthRetryError`
- The surface repository mapper, for party creation
  `PartyRepository._mapAuthSessionExpired`

The canonical user surface is the existing `AUTH_SESSION_EXPIRED` banner with
user-driven Retry. Do not add an automatic repair/retry from repository error
exhaustion.
