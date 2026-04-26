---
name: Evidence gates apply to production hardening, not dead-code cleanup
description: Require evidence before shipping or reverting production hardening. Do not use evidence gates to preserve zero-consumer code or block small direct UI/source changes.
type: feedback
status: scoped-rewrite
originSessionId: 29573519-30ff-4263-9565-9f2fa2074e35
---

**Rule:** Shipping, relaxing, or reverting production hardening requires evidence. Deleting zero-consumer source code, removing unused UI branches, or shipping a small direct UI/source change does not require a telemetry gate.

**Why:** The original rule prevented doctrine flips based on vibes after ACL incidents. That remains correct for production hardening. It becomes harmful when applied recursively to source-level orphans or UI simplification, where there is no production traffic to measure and the carrying cost is comprehension debt.

**How to apply:**
- For auth, billing, ACL, RLS, payment, webhook, and data-plane hardening, cite telemetry, probes, incidents, or verified shipped-code facts before changing posture.
- For raw ambiguous error rates, bucket the evidence before making a doctrine flip.
- For zero-consumer code, delete now per `feedback_no_consumer_delete_now.md`.
- For UI work, ship direct unless the user explicitly asks for a flag.
- If an audit says "keep X until evidence," first classify X: reachable production defense, or source/UI orphan. Only the first gets an evidence window.
