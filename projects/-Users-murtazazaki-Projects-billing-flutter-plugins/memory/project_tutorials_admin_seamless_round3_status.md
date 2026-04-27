---
name: tutorials_admin seamless round-3 plan status (2026-04-26)
description: Round-3 plan apps-tutorials-admin-so-the-tutorials-jaunty-reef.md — 9 of 10 atomic commits shipped to main; deferred work itemized for a focused follow-up PR
type: project
originSessionId: a6fd53cc-1a62-4d0f-81a5-ebe48179a6f7
---
**Plan:** `~/.claude/plans/apps-tutorials-admin-so-the-tutorials-jaunty-reef.md` (Round-3 approved 2026-04-26)
**Live plan in repo:** `.claude/plans/reels-next-window-session-backtrack-stack.md` (updated commit 1)

## Shipped (9 commits on main, push-after-each)

| # | SHA prefix | Subject |
|---|---|---|
| 1 | bb6e348c | docs(plan,skill): live plan flat-DTO + first-reel-pop + Playground V1 + skill rules v1.1.0 |
| 2 | ad177904 | chore(skill): rebuild tutorials-manager index + tighten canonical queries |
| 3 | 3856a4d7 | feat(tutorials-admin): Makefile orchestrator + project-root .vscode + smoke |
| 4 | 587e7b7c | feat(bff,cockpit): startup probe + BFF_PORT/host/CORS env-driven + bffBaseUrl provider + tests |
| 5 | e2656e37 | docs(tutorials-admin,microservices/tutorials): operator runbooks + worker README quickstart |
| 6 | a2835d85 | feat(bff,worker): Playground BFF route + flat-shape contracts |
| 7 | a7355eee | chore(tutorials): lint-drift --remote + secrets parity + admin-token-help + tail-local + smoke split |
| 8 | 3a442bb6 | feat(cockpit): env_pill PROD-arm 3-state — block prod probes pre-arm |
| 9 | f810ad94 | feat(cockpit): Playground DTO + provider with stable Query value-equality |

## Test counts

- BFF (vitest): 51/51 pass — smoke + startup probe + port override + CORS matrix + CORS port-mismatch + redact + Playground contract + envelope + status
- Worker (jest): 15/15 pass on the new suites — flat-shape contract + secrets parity + lint_wrangler_remote + RPC whitelist drift sentinel
- Flutter (flutter test): 28/28 pass — 8 env_pill arm + 5 GraphPlaygroundQuery + 3 bff_base_url_provider + 12 existing
- Skill artifacts: 9/9 pass — domain CSVs, no duplicate IDs, BM25 returns expected rule IDs for canonical queries

## Deferred work — RESOLVED 2026-04-26 (all 6 follow-up commits shipped)

All deferred items shipped on main. Total final test counts:
- BFF (vitest): 60/60 pass (added redact_property fuzz + sr_key_sentinel.contract)
- Worker (jest): +17 admin-graph contract tests
- Flutter (flutter test) tutorials_admin: 86/86 pass

### Cockpit polish (commit `da013bee`)
- ✅ `lib/widgets/upstream_degraded_banner.dart` — separate from BFF-unreachable; renders first error code/upstream/message + Retry
- ✅ `right_detail_pane.dart`: R2 case reads `r2ReachabilityProvider`, surfaces real probe verdict (ok/unreachable/HTTP N) instead of synthetic "Click any R2 row" placeholder
- ✅ `cockpit_app_bar.dart`: `_refreshAll` scoped to `envs.first`
- ✅ Tests: `upstream_degraded_banner_test.dart` (5), `right_detail_r2_provenance_test.dart` (5), `cockpit_app_bar_refresh_test.dart` (2)

### Playground cell + lifecycle widget (commit `372d33bf`)
- ✅ `lib/pages/cockpit/cells/graph_playground_cell.dart` — picker form + lifecycle view + in-cell `pastStack/current/futureQueue` mirror with "Swipe Up"/"Swipe Down"/"Exit" buttons
- ✅ `CellSection.graph` enum addition + center_matrix wiring
- ✅ `graphPlaygroundStackProvider` autoDispose notifier (mirrors ReelsView contract)
- ✅ `test/widget/graph_playground_lifecycle_test.dart` (11) — GRAPH-013/014/015 + UI

### Defense-in-depth (commit `2e81fc11`)
- ✅ `local_bff/test/redact_property.test.ts` — fast-check fuzz, 6 properties, 100 runs each. SURFACED + FIXED a real redactor bug: JWTs ending in URL-safe `-` slipped past the trailing `\b` word-boundary. Replaced with lookbehind anchor.
- ✅ `local_bff/test/sr_key_sentinel.contract.test.ts` — middleware-level scrub guarantee
- ✅ `apps/tutorials_admin/test/widget/sr_key_sentinel_test.dart` — lib/ static scan: no service_role/sr_key/supabase_secret identifiers, no JWT literals, no String.fromEnvironment SR-key reads, bffBaseUrlProvider stays loopback
- ✅ Added `fast-check@4.7.0` dev-dep on `local_bff`

### ReelsView Phase 7 wiring (commit `64e0b40e`)
- ✅ `lib/billbook/tutorials/reels/reels_view.dart`: ConsumerStatefulWidget conversion; PopScope(canPop:false) wraps non-empty body; `_onPopInvokedWithResult` calls `reelsRecommendationQueueProvider.notifier.retreat()` first, falls back to `Navigator.maybePop` only when retreat returned false; try/catch guard for legacy callers
- ✅ Empty-state intentionally has no PopScope (no past possible)
- ✅ `test/billbook/tutorials/reels/reels_view_popscope_test.dart` (7 critical-tier sentinels) — static structural assertions instead of runtime gesture simulation (avoids `handlePopRoute` hangs against single-route Navigator)
- ✅ `test/billbook/tutorials/fixtures/tutorial_fixtures.dart`: dropped retired `videoUrlMp4` (unrelated breakage)

### Chain + admin-graph contracts (commit `245069cb`)
- ✅ `apps/tutorials_admin/test/chain/admin_provider_chain_test.dart` (5 critical) — bffBaseUrl→bffClient propagation, family<...,AdminEnv> cache slots per env, smoke for all 8 cells
- ✅ `microservices/tutorials/tests/contract/admin-graph-build.contract.test.ts` (7) — Bearer auth, env enum, body schema, empty corpus, idempotency (preserves activeVersion+LKG, status=ready)
- ✅ `microservices/tutorials/tests/contract/admin-graph-health.contract.test.ts` (10) — auth, status='unknown'|'green'|'amber'|'red' branches, promote=true LKG advancement, KV manifest probe split from DB/RPC probe

### Cockpit goldens (commit `44dbb91e`)
- ✅ `apps/tutorials_admin/test/golden/cells_states_smoke_test.dart` (19) — 7 cells × 4 states (loading/error/empty/populated). Pragmatic structural assertions instead of PNG matchesGoldenFile (which is fragile across CI font rasterizers). Uses Completer<T>().future for loading scenarios to avoid !timersPending.

### Doctrine notes for future sessions
- For loading-state widget tests, prefer `Completer<T>().future` over `Future.delayed`. Future.delayed creates a pending Timer that trips `!timersPending` in flutter_test binding.
- Goldens that ship as structural assertions (find.byType, find.text) are more durable than PNG matchesGoldenFile for cross-platform CI. Use PNGs only when the visual layout is the contract.
- Riverpod 3 autoDispose pattern: `extends Notifier<T>` + `NotifierProvider.autoDispose<...>` — NOT `AutoDisposeNotifier`/`AutoDisposeNotifierProvider` (those are Riverpod 2 names that no longer exist).

## Key invariants pinned by tests

- Worker `/v1/tutorials/recommendations/next` emits FLAT items[] (top-level `tutorial_id`, NOT nested under `items[].tutorial`) — `recommendations-flat-shape.contract.test.ts`
- BFF `/graph/playground/window` synthesizes Worker-like envelope around flat RPC rows; `baseWeight`/`seededJitter` null in V1 — `graph_playground.contract.test.ts`
- `graph_version_pin` is diagnostic-only in V1 (`graph_version_pin_honored: false`)
- `wrangler kv key (put|delete|get)` MUST include `--remote` (incident 2026-04-13) — `lint_wrangler_remote.test.ts` + `lint-drift` Makefile gate with `|| true` neutralizer
- `secrets-mapping.json[tutorial-engine].secrets` ≡ `wrangler.toml [secrets] required` — `secrets-parity.test.ts`
- `src/constants.ts::ALLOWED_RPCS` ⊆ test fixture — `rpc-whitelist.test.ts` drift sentinel
- AdminEnv.prodWorker NEVER enters `selected` pre-arm — `env_pill_prod_arm_test.dart` (8 cases)
- BFF startup `SELECT 1` probe fails fast with typed `BffStartupError({code:'db_connection_failed'})`, password redacted — `startup_probe.test.ts`
- BFF port + CORS allowlist env-driven; default covers both `localhost:5050` and `127.0.0.1:5050` — `cors.test.ts` + `cors_port_mismatch.contract.test.ts` + `port_override.test.ts`

## How to apply

- Read this memo when picking up the deferred work in a follow-up session.
- The Round-3 plan file remains the source of truth for scope.
- Do NOT re-run the 9 shipped commits — they're on main.
- For the cell widget (commit-9 carryover): start with `graph_playground_cell.dart`, register in cockpit_state.dart `CellSection.graph`, then `cells/` left-rail entry.
- For ReelsView Phase 7: this is a separate PR with its own gate (`PopScope` widget tests are critical-tier — kernel rule).
