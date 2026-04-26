---
name: Retire ADRs by archival/trim, not append-only amendments
description: When an ADR is being retired or superseded, don't grow the file with multiple Post-Incident Updates; archive or trim the file and put incident learning in memory.
type: feedback
originSessionId: e4a3da24-0e58-4bd3-8954-436f09de5b1a
---
Don't append multiple `## Post-Incident Update #1`, `#2`, … sections to an ADR that's being retired. If a decision is retired, the ADR file should be archived (moved to `docs/decisions/archive/`) or trimmed to its essential supersession pointer — not grown further. Live ADR files stay terse and current; retired ADRs don't clutter the live docs/decisions/ directory as context load.

**Why:** 2026-04-23 — operator rejected v4 plan's proposal to append ADR-009 §Post-Incident Update #2 on top of the earlier Post-Incident Update. Reason: *"If something is being retired and not going to be used, it's fine to be as a memory of what went wrong, but just kind of free up the context road a bit."*

**How to apply:**
- Retiring an ADR → move the file to `docs/decisions/archive/` OR trim it to a supersession pointer + date.
- Incident learning (what went wrong, root cause, arc of migrations) → save to `.claude/.../memory/` as a `project_*.md` entry.
- Rules codified from the incident → add to the right skill CSV at organize-time, not during P0 ship.
- Live `docs/decisions/` directory should contain only currently-authoritative ADRs, each one terse. Growing retired ADRs creates context-drag on every subsequent session that reads the directory.
