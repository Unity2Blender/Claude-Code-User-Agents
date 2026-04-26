---
name: Probe policy — use existing endpoints, never invent
description: When designing a Phase 0 diagnose, audit the live routes BEFORE inventing endpoints. Use health + ONE real authenticated route + diagnose only on disagreement.
type: feedback
originSessionId: 36039f2b-32bf-4fe5-849f-bc2739e79e2d
---
When designing a "diagnose" phase for an incident response or pre-cutover audit, **do not invent endpoint names**. Audit the live Worker routes first.

**The incident:** Plan draft (2026-04-22) referenced `/v1/admin/banner-pool-debug` as the diagnostic endpoint. That endpoint **does not exist**. The live diagnostic route is `/v1/tutorials/banner/diagnose/:placement` (per `microservices/tutorials/src/routes/banner-diagnose.ts`).

The plan also assumed `/v1/tutorials/config` had a 300s TTL. Live verification (`config.ts:105`) shows `Cache-Control: public, max-age=60` — 60s, not 300s. Rollback SLA estimates were 5x off.

**The right probe policy:**

1. **Health probe** (`/health/banner/:placement?cohort=...`) — fast, no auth, but doesn't fully mirror real traffic (skips KV hit semantics, different timeout).
2. **ONE real authenticated route** (e.g., `/v1/tutorials/banner?placement=...&build_number=...`) — captures actual KV behavior and real latency.
3. **Diagnose** (`/v1/tutorials/banner/diagnose/:placement`) — ONLY when the health probe and the real route disagree. Diagnose returns 7 independent reads in one JSON blob (raw PostgREST, both legacy + graph RPCs, KV state, placement_config row, source_edges, active_resolver_path, git SHA).

**Why this order:** Each probe is more expensive than the last. Health is cheap; auth-route adds JWT cost; diagnose hits 7 backend systems. Run them as a triage cascade, not in parallel.

**How to apply:**
- Before designing any Phase 0 diagnose, run `ls microservices/<svc>/src/routes/` to see what actually exists.
- Verify TTLs by reading the actual `Cache-Control` header expression in the route file, not by recall.
- Document the probe sequence in the runbook so on-call doesn't reinvent it.

The plan that surfaced this: `image-1-docs-runbooks-lib-billbook-tuto-ethereal-sparrow` (2026-04-22, F4 + F5 + D1).
