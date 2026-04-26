---
name: Observability before risky cutover, sized to blast radius
description: Risky data-plane or contract cutovers need evidence before cutover, but the evidence channel should be the smallest useful one.
type: feedback
status: scoped-rewrite
originSessionId: ae1c4a54-b698-4ebb-a33e-67a6d5fbe956
---

**Rule:** Before a risky data-plane, schema, API, auth, billing, or cross-service contract cutover, ship the smallest evidence channel that can verify the cutover and diagnose failure.

**Why:** The YC water-test philosophy only works if leaks are visible. But observability is not a license to overbuild. For many changes, structured logs or counters are enough. Health endpoints, synthetic checks, Logpush, dashboards, and materialized telemetry tables are warranted only when the blast radius or operational need justifies them.

**How to apply:**
- For schema/API/auth/billing cutovers, include structured outcome evidence before or with the cutover.
- Start with the smallest evidence: logs or counters with reason buckets.
- Add health endpoints and synthetic probes when operators need an external green/red signal.
- Add dashboards or durable aggregation when the decision requires trend data or repeated operator review.
- Do not require observability-first ceremony for small localized UI/source edits.
- Do not add UI flags to satisfy observability posture unless the user explicitly asks.
