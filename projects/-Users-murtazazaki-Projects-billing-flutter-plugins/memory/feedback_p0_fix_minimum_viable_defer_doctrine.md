---
name: P0 fix = minimum viable; defer rule codification + context reorganization
description: On P0 emergencies, ship the fix alone. Rule CSVs, ADR amendments, new critical tests for doctrine, manifests, edge-function observability — all deferred to follow-up plans.
type: feedback
originSessionId: e4a3da24-0e58-4bd3-8954-436f09de5b1a
---
When an incident is P0 (paying users bleeding), the fix plan must contain ONLY what's load-bearing for closing the fire. Rule codification, ADR amendments, new pgTAP tests that encode doctrine, manifests, interim observability edge-functions — all belong in follow-up plans, not the emergency PR.

**Why:** 2026-04-23 — v4 plan for 000363 bundled the SQL migration with: 4 new pgTAP files (ACL-003, BHVR-025, TEST-TIER-004, body-gate return-value), 6 new rule CSV entries, manifest file + validate script, rollout analytics event, ADR §Post-Incident Update #2. Operator rejected the bundling: *"First, implement the fixes, then we will be organizing the context and rules and stuff."*

**How to apply:**
- P0 PR: migration + MINIMUM tests to prove the fix holds (ideally one positive invariant) + operator-visible ADR status flip (retire/supersede via header field or archival).
- Deferred to follow-up plans: rule CSV additions, new pgTAP tests that encode general doctrine (not THIS incident's invariant), manifests, observability tooling, ADR rewrites, memory entries beyond one `project_*.md` incident record.
- The follow-up plan can then take its own audit + test cycle without blocking the prod fix.
- Exception: rules where the post-condition IS the test that proves the fix worked (e.g., migration's own DO-block assertion) stay in the P0 PR.
