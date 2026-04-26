---
name: 3-layer impact map for plan audits
description: Plan audits must trace every changed type/column/handler through Flutter + Worker + SQL with file:line — agent-only audits miss reverse-dependency edges
type: feedback
originSessionId: d9f48ca5-88b8-449d-ae84-c7bfa9c170d6
---
When auditing a plan that changes a type, column, handler, or contract, the audit MUST produce a 3-layer impact map citing file:line for **each** layer:

1. **Flutter** — model fields, providers, dispatch handlers, UI render paths, test fixtures
2. **Worker / Edge** — endpoint reads, filters, allowlists, response shape, contract tests
3. **SQL** — column, CHECK constraint, trigger, RPC body, related FK, view, migration history

**Why** (2026-04-25 incident): A plan to retire the `reel-watch` CTA type and drop `cta_config_override` + `video_url_mp4` columns was approved by 2 audit agents (`product-architecture-lead` + `visual-designer-lead`, covering 5 skills). The operator's own Ultraplan refinement caught:

- `reel-watch` still live in **4 places** (Flutter `cta_resolver:319-390`, Flutter `reel_video_card:486-493`, Worker `schema-constants:58-67`, SQL `000365:97-123`); agent audit cited 3.
- `cta_config_override` reads in Worker `banner.ts:83-84, 157-195` AND SQL `COALESCE(b.cta_config_override, t.cta_config)` in `000357:85-88`; agent audit didn't trace the SQL RPC body.
- `video_url_mp4` reads in Flutter pool (`reels_controller_pool:31-41`) + Flutter render branch (`reel_video_card:562-575`) + Worker `tutorial-by-id.ts:41-71`; agent audit cited only 1 site.
- Compat view `public.tutorials` still depended on by 4 graph migrations (000305:25-26, 000306:25-26, 000307:79-82, 000312:35-37) — agent audit treated it as imminently droppable.
- `UNIQUE (tutorial_id, placement_id)` constraint from 000261:67-76 entirely absent from the audit.

**How to apply**: For every "drop X" or "retire X" or "change X contract" decision in a plan:

1. **Spawn an Explore agent specifically for the reverse-dependency map**: prompt is "Find every read/write of X across `lib/`, `microservices/`, `supabase/migrations/`, `supabase/tests/`, `apps/`. Cite file:line. Do not summarise — enumerate."
2. **Block plan finalisation** until the impact map is in the plan body (not just an audit appendix).
3. **For columns being dropped**, also enumerate: triggers watching the column, CHECK constraints referencing it, RPC bodies SELECT'ing or COALESCE'ing it, GENERATED columns deriving from it.
4. **For views being dropped**, list every FK and migration that references them; PostgreSQL silently auto-rewrites FKs through view-renames but FAILS on view-drops if dependents exist.
5. **Treat the operator's Ultraplan refinement pass as a HARD gate**, not a soft suggestion (per memory `feedback_ultraplan_refinement`). Never ExitPlanMode until the operator's own audit clears the plan.
6. **For "retire CTA type" / "simplify endpoint" decisions**, ALWAYS check if the endpoint is computing something the client should compute (e.g., banner returning `effective_cta` is an SRP smell — banner is a row, not a CTA computer).
