---
name: Seibel water-test means ship small plus targeted evidence
description: Turn on the water with the smallest reversible change and bucketed evidence. Do not convert the principle into speculative toggles or heavyweight observability for every change.
type: feedback
status: scoped-rewrite
originSessionId: 29573519-30ff-4263-9565-9f2fa2074e35
---

**Rule:** A production water-test ships the smallest reversible change that can expose the weak pipe, plus targeted evidence that makes leaks diagnosable.

**Why:** Raw error rate creates panic and arguments from vibes. But requiring a full telemetry pipeline for every change creates its own drag. The useful middle is bucketed evidence sized to the risk: structured reason fields first, durable counters/dashboards only when multi-day aggregation is actually needed.

**How to apply:**
- Prefer small direct releases over broad speculative hardening.
- Add bucketed evidence in the same PR when the rollout can produce ambiguous production failures.
- Use structured logs with dimensions such as reason, endpoint/function, role, user hash, placement, build number, or auth state where relevant.
- Add durable tables, cron aggregation, health endpoints, or dashboards only when the decision depends on multi-day trend analysis or operator-facing health state.
- Do not add UI runtime flags as a substitute for water-testing. UI flags require explicit user request.
- When asked whether to revert based on raw logs, first ask what bucket dominates. If there is no bucketed evidence and blast radius is real, add the smallest useful bucketing before making a doctrine flip.
