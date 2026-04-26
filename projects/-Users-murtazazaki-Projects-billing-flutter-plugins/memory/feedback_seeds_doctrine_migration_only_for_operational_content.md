---
name: Seeds Doctrine — migration-only for tutorials/banners/reels/edges
description: Seeds run once per db reset. Operational content (tutorials/banners/reels/edges/graph) ships via migrations so `migration up` works steady-state.
type: feedback
originSessionId: 83fb8343-bb17-4517-a0c9-061d55c22211
---
Seeds run ONCE per `db reset`. After that, only migrations matter. Operational content that needs to evolve must ship via migrations, not seeds.

**Why:** Per the user: *"For tutorials, we don't have any seeds. We don't want to maintain any seeds, be it for tutorials, tutorial banners, tutorial reels, tutorial edges, and this entire graph thing. We manage it via migrations because we can do migration ups and stuff."* Coupling operational content to seeds means it only updates on `db reset` — incompatible with the Local Docker Policy of `migration up` first.

**How to apply:**
- Tutorials, tutorial_banners, tutorial_reels, tutorial_reel_edges, tutorial graph data → migration-only (INSERT/UPDATE/DELETE statements inside numbered migrations).
- Reference data (HSN, districts, pin_codes) → seed file is acceptable since these change rarely and `db reset` is rare.
- Customer/tenant config (feature flags, CTA targets, banner placements) → migration-only when row count is small (<1k) and changes meaningfully; seed when bulk-loaded reference.
- For rebaseline: when extracting reference/config rows into a baseline, use literal `INSERT ... VALUES` from `pg_dump --data-only --table=<name>`, not `INSERT INTO ... SELECT` from linked (the baseline applies on a fresh local with no link).
- This rule lives as `SEED-001` in `.claude/skills/design-postgres-tables/data/rules/`.
