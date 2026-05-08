# Feedback: PostgREST SWEEP RPC Classification

Date: 2026-05-06
Status: Active

PostgREST routes Firebase JWTs missing `role: "authenticated"` to the configured
anonymous database role. In Supabase that role is `anon`. The logged
`authenticator` role is the NOINHERIT connection role; client traffic should
not rely on grants to it because PostgREST switches away before execution.

Classification rule:

- SWEEP: SECURITY DEFINER public RPC has an early null-auth body guard and then
  extracts identity through `firebase_uid()`, `has_billing_access`, or
  `user_billing_id`. Grant `anon`, `authenticated`, and `service_role`; keep
  `authenticator` and direct `PUBLIC` ungranted.
- CLOSED: RPC has no body-level JWT identity extraction or must fail before
  body execution. Grant only `authenticated` and `service_role`.

Incident evidence: `000444` revoked anon EXECUTE on `upsert_party`; `000456`
completed CLOSED hardening; April-4 clients missing the role claim then failed
with planner-stage 42501. `000457` restored `upsert_party` to SWEEP.
