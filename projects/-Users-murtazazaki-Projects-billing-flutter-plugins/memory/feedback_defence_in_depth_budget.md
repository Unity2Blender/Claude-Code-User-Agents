---
name: Defence-in-depth has a YAGNI budget
description: Each defence layer must earn its cost with a DISTINCT failure mode it uniquely catches. Remove layers whose catch rate is measurably zero for 30 days.
type: feedback
originSessionId: 29573519-30ff-4263-9565-9f2fa2074e35
---
**Rule:** Layers of defence are YAGNI-taxable. For billing-gated surfaces specifically, the chain is: (1) ADR-001 client-transport guard, (2) body-gate via `has_billing_access()` in SECDEF function bodies, (3) server-side ACL revoke on `anon` (ADR-009 §Decision). Each layer must earn bundle/latency/ops cost with a distinct failure mode it uniquely catches. If the cumulative rate of "cases Layer N alone catches" = 0 for ≥30 days, delete Layer N and keep the rest.

**Why:** 2026-04-23 000360 incident — Layer 3 (ACL revoke) ALONE catches "body-gate has an implementation bug AND transport-guard has a leak AND the function body got renamed without updating `has_billing_access` calls." All three AND'd together = very low rate. Yet Layer 3's own implementation bug (missing GRANT-to-authenticated in 000360) produced a 24h P0 with ~72 functions broken for paying users. Cost multiplier of the layer's own failure mode was ∞/0 vs its distinct catch rate.

Symmetric framing: Layer 3 isn't wrong, it's on a budget. Post-ORCH-001 (7-day bucketing), if `role_mismatch` counter is 0 while `broken_authenticated` is non-zero, Layer 3 is earning nothing and costing refactors. If `role_mismatch` > 0 with clear Layer 1/2 leak evidence, Layer 3 keeps its earn.

**How to apply:**
- Before ADDING a defence layer, specify: what distinct failure mode does it catch that existing layers don't? Expected rate? Cost of adding? Cost of removing later?
- Before DELETING a defence layer, specify: what's the measured rate of "cases only this layer catches" in the last 30 days? If zero — good candidate for removal.
- If you can't answer either, ship observability FIRST, measure, then decide.
- Apply specifically to MATPU / billing ACL chains. Don't over-generalise to all defence-in-depth (UI retries, client-side validation have different cost curves).
