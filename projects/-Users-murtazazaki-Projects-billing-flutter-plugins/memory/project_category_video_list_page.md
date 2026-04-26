---
name: Category Video List Page
description: New page between category grid and ReelsView — 29 locked decisions, icon registry, watch-state endpoint rename, per-video icon_config JSONB
type: project
originSessionId: 52a8b03f-a491-467e-b2e2-42e0a968ed1c
---
Category Video List Page added to `lib/billbook/tutorials/category_list/` (v5.3).

**Why:** Users had no way to browse video sequences or see progress — tapping a category jumped straight to ReelsView.

**29 locked decisions (D1-D19 from round 1, D20-D29 from round 2):**
- Per-video `icon_config` JSONB + category fallback (D1)
- Material-only icons via const registry map (D5) — preserves tree shaking ~1.6MB
- Checkmark + NEW badge, no progress bar (D7)
- Hero card hidden for first-time/anonymous users (D12)
- Single video → skip list, go straight to ReelsView (D16)
- **GET endpoint renamed** from `/v1/tutorials/views` to `/v1/tutorials/watch-state` (D20)
- POST stays at `/v1/tutorials/views` — already deployed, no rename (D21)
- Server-first badges via GET endpoint, not local-only (D22)
- Response shape: `{completed, total_watch_seconds}` only — `watch_count` dropped (D23)
- Optimistic UI: markSeen() immediately + background server refetch (D24)
- Keep dual providers: local `tutorialSeenProvider` for grid, server for list (D25)
- Handler file: new `watch-state.ts` separate from `views.ts` (D26)
- `VideoWatchState` model: `watchCount` field removed (D27)
- New migration 000239 to patch 000238 (D28)
- Shared cache key constant in `src/utils/cache-keys.ts` (D29)

**Schema:** M000237 (`icon_config` JSONB), M000238 (`get_user_tutorial_views` RPC), M000239 (trim `watch_count` + add `PARALLEL SAFE`)
**Worker:** `GET /v1/tutorials/watch-state` endpoint (Phase B — not yet implemented). Handler: `watch-state.ts`, registered with `requireReadsEnabled`.
**Tests:** 20 tests across 3 files (audit found 21+ more needed: provider chain, tile/card widget, edge cases)
**Audits:** 9 DevOps findings (WK-CRIT-01 resolved by rename), 26 Flutter/Postgres/Test findings

**How to apply:** Use Amendment v2 in the plan file for implementation. Critical: register GET with `requireReadsEnabled` (not `requireWritesEnabled`). Add `get_user_tutorial_views` to `ALLOWED_RPCS`. Add `watch-state:` to purge "all" pattern.
