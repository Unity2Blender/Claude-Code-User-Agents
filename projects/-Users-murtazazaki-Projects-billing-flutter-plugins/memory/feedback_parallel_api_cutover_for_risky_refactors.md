---
name: Parallel API cutover for risky refactors
description: Ship new runtime as additive parallel API; defer big-bang migration to explicit-confirm task when blast radius >50 prod refs
type: feedback
originSessionId: 88479857-ca35-4a9f-bd6b-ef7c16840ff0
---
When a refactor touches >50 production refs OR >1000 LOC of dispatch / coordinator code, ship the new API as a PARALLEL surface (additive, zero-breakage) and DEFER the migration cutover to a separate explicit-confirm task.

**Why:** Phase 5B-L (CTA two-surface refactor) was scoped as a single big-bang change in the §v8 spec. Audit (Phase 5B-A) revealed 79 production CtaGateResult refs across `lib/` and `apps/gst_calculator/lib/` plus 1171 LOC of dispatch code (cta_resolver.dart 516 + feature_navigation_coordinator.dart 655). Per Auto mode safety rule, that warrants explicit user confirmation before the cutover, NOT autonomous execution.

**How to apply:**
1. Foundation classes (Phase 5A) — additive, ZERO breakage. Auto-execute.
2. API expansion (Phase 5B-A) — additive variants, additive descriptor field. Auto-execute.
3. New runtime ships PARALLEL to legacy (Phase 5B-B) — both work side-by-side. Auto-execute.
4. Migration cutover — DEFERRED. Requires explicit operator confirmation + device testing because production callers see behavior change.
5. Coordinator refactor — DEFERRED for same reason.

**Audit cadence:** before any refactor task spec is final, count production refs (`grep -rn '<symbol>' lib/ apps/`) and LOC of files touched. If either crosses threshold, propose phasing in audit synthesis BEFORE writing the spec.

**Phase 5B-A audit added 8 directives the original §v8 spec missed:**
GATE-OUTCOME-PARITY-001, REGISTRY-DUAL-OWNERSHIP-001, REELS-BROWSER-DUAL-CLASS-001, GATE-SIDE-EFFECT-AUDIT-001, PARALLEL-API-CUTOVER-001, CACHE-SEMANTICS-LOCK-001, AST-SENTINEL-LIGHTWEIGHT-001, DESCRIPTOR-MIGRATION-ORDERING-001.

**Cutover trigger condition:** when (a) operator explicitly authorizes the cutover task, (b) device-test campaign on critical CTA paths is scheduled, (c) the parallel API has soaked for ≥1 release cycle without issues found via tests.
