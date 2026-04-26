---
name: Test Suite Hardening v1 (GST Calculator Launch Gate)
description: Test hardening plan executed April 2026 — 8 workstreams, 280+ new tests, FakeSupabaseClient infrastructure, Glados PBT
type: project
originSessionId: 5a9b6f59-f38b-429f-ad50-ecb378ea0af8
---
April 2026 test hardening targeting GST Calculator launch gate.

**Why:** Audit found 25.8% manifest coverage, zero tests for core utils (round2, currency, HSN), zero FSM/contract tests, 40+ files violating MOCK-PROV-003 (mocking PostgREST directly), and 5 dead Firestore adapters.

**How to apply:**
- All new tests use `FakeSupabaseClient` (test/helpers/fake_supabase_client.dart) — never `MockPostgrestClient`
- GST calculations use Glados property-based testing (invariants, not just examples)
- DTO models with `.fromSupabase()` need sentinel tests (catches schema drift)
- Subscription provider has FSM tests (guards, recovery, terminal states)
- Directives D-1 through D-7 in plan file govern ongoing test quality

**Key artifacts:**
- Plan: `.claude/plans/wobbly-crunching-cerf.md`
- FakeSupabaseClient: `test/helpers/fake_supabase_client.dart`
- Glados GST tests: `test/core/utils/gst_property_test.dart`
- Subscription FSM: `test/state/providers/subscription/subscription_fsm_test.dart`
- Chaos tests: `test/chaos/network_interruption_test.dart`
