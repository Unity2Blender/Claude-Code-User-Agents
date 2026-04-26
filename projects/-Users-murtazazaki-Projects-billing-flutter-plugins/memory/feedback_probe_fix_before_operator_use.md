---
name: Fix the probe before using it as verification
description: If the verification probe is the broken component, fix + deploy BEFORE treating its output as a go/no-go gate
type: feedback
originSessionId: e2f8e44b-274b-414f-a32e-6a5f6b0e99b4
---
When an incident investigation reveals that the probe/health endpoint itself is returning incorrect results, the correct sequence is **fix → deploy → verify**, not **purge → deploy → verify-via-broken-probe**.

**Why:** A broken probe lies in both directions. It can say "unhealthy" when prod is fine (false-negative — causes wasted rollbacks) OR "healthy" when prod is broken (false-positive — lets bad state go undetected). Using it as a cutover gate is worse than having no probe at all.

**How to apply:** When designing or auditing a cutover sequence, ask: "Is the probe I'm using for verification the same one showing the symptom?" If yes, fix the probe first. Ship the fix before any other production change (KV purge, flag flip, migration apply). Only then run the cutover, using the now-trustworthy probe as the gate.

**Incident that seeded this rule:** 2026-04-22 banner steady-boot residuals. `/v1/admin/health-data-sentinels` reported `healthy:false, actual:0` due to a client-side-filter-after-limit truncation bug (see `project_gst_sentinel_probe_clientside_filter_truncation.md`). An earlier draft of the cutover plan had "KV purge → deploy → verify via broken probe" — useless sequence because the probe can't tell you whether the purge worked. Audit caught it; sequence was corrected to "fix probe → deploy → purge → verify".

**Adjacent pattern:** probe routes should also have integration/contract tests that seed the exact bug condition. Locks the regression at the test layer so "fix the probe" doesn't re-break on a future unrelated edit.
