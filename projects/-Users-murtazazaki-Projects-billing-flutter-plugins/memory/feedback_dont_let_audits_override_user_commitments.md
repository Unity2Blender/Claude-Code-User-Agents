---
name: Don't let audit agents override explicit user commitments
description: Process feedback — audit findings are PROCESS feedback, not commitment renegotiators; when audit says "stub it" but user explicitly committed, user wins
type: feedback
originSessionId: 1636fa18-93d5-4e42-b373-51c0d539de25
---
**Rule:** Audit agents (flutter-architecture-principles, design-postgres-tables, devops-architect, brainstorming, etc.) provide PROCESS feedback and technical correction. They do NOT have authority to renegotiate decisions the user has explicitly committed to in the interview round. When audit critique suggests softening/walking back a user's commitment, preserve the commitment and carve out the audit's legitimate concern (usually about scope/session boundary) as a separate constraint.

**Why:** On 2026-04-22 during ADR-007 plan authoring, the brainstorming audit flagged "ADR-008 (pool-cohort RPC) is pre-committed in a painkiller plan — soften to stub." I (Claude) absorbed this as "ADR-008 is not a commitment." Plan v2.2 reduced ADR-008 from committed follow-up to open-ended stub. Operator caught this and corrected via ExitPlanMode rejection: LD-3 was explicitly "Hybrid: Harden per-placement now + pool-RPC follow-up" — the follow-up IS locked. The audit's valid observation was that ADR-008 needs its own brainstorming session (session scope), NOT that the commitment itself was renegotiable.

**How to apply:**
1. When an audit report says "walk back X" and X was a locked decision from the interview round, preserve X. Carve out the audit's technical critique (usually about implementation/scope/session) as a separate addendum.
2. Operator's interview answers in AskUserQuestion rounds are LOCKED unless they explicitly reopen the decision. Subsequent audit critiques are advisory on implementation, not veto power over the decision.
3. When writing plan revision logs, distinguish "audit correction" (valid technical fix) from "audit walk-back" (attempt to renegotiate operator commitment). The first ships; the second gets pushed back.
4. Plan v2.2 restored ADR-008 with the 4 locked bullets verbatim. Plan v3.0 preserves this restoration. ADR-008 stub lives at `docs/decisions/ADR-008-banner-pool-cohort-rpc.md` with explicit "COMMITTED follow-up per LD-3" framing.

**Source:** ADR-007 plan v2.2 → v3.0 operator refinement, documented in §14 revision log. Karpathy Comprehension-Debt discipline extended to "don't let audits do the thinking about what the user agreed to."
