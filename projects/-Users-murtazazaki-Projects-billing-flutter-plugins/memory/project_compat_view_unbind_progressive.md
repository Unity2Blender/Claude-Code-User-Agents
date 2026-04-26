---
name: Compat-view unbind progressive (tutorials → tutorials_reels)
description: When the public.tutorials → tutorials_reels rename ships with a backward-compat view, any RPC body you touch in a coupled-deploy slice should be re-issued with FROM public.tutorials_reels direct. Don't defer.
type: project
originSessionId: 36039f2b-32bf-4fe5-849f-bc2739e79e2d
---
**The hazard:** Migration 000341 renamed `tutorials` → `tutorials_reels` and created a backward-compat view `public.tutorials AS SELECT * FROM tutorials_reels`. The view is scheduled to drop ≥7 days post-Worker cutover per the 000341 contract.

**Multiple RPCs and Worker probes still read the view:**
- `get_banner_recommendation` v3 (000250) — banner pool
- `get_video_playlist` (000238) — swipe-next playlist
- `get_tutorial_by_id_v2` (000298) — single tutorial lookup
- `get_tutorial_by_id_v3` (000313) — newer single tutorial lookup
- Worker `microservices/tutorials/src/routes/health.ts:426-435` — health probe

When the view drops, every one of these breaks unless its body is re-issued with `FROM public.tutorials_reels` direct.

**The discipline:** Any RPC body you touch in a coupled-deploy slice MUST be re-issued with `FROM public.tutorials_reels` direct (drop the view dependency). Don't defer to "we'll fix this later when the view drops" — that's how the view-drop migration lands and 4+ RPCs fail simultaneously.

**How to apply:**
- Plan-time: when scoping any change to tutorial RPCs, audit which migrations + Worker routes still read `public.tutorials`. Either include them in this plan's scope or explicitly call out the deferred ones in `## Out of Scope`.
- Migration-time: when issuing a new RPC version (DROP+CREATE), the new body MUST use `FROM public.tutorials_reels`, never `FROM public.tutorials`. Single source of truth.
- Test-time: pgTAP critical tests that assert RPC behavior should also assert `pg_get_functiondef(...) NOT LIKE '%FROM public.tutorials %'` (negative match) to prevent regression.

The plan that surfaced this: `image-1-docs-runbooks-lib-billbook-tuto-ethereal-sparrow` (2026-04-22, F3 + D10). Phase 1.5 unbinds the 2 banner-relevant RPCs; the remaining 2 (`get_tutorial_by_id_v2/v3`) and `health.ts` are flagged as out-of-scope follow-ups in plan `worker-tutorials-view-drop-coordinated-cutover-non-banner-rpcs`.
