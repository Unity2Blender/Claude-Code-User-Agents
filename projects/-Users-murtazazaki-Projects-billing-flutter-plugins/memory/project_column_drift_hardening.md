---
name: Column-drift hardening plan (2026-04-12 42703 incident)
description: 7 locked decisions + Phase 0 shipped for systemic SQLSTATE 42703 observability across Flutter/Supabase. Plan at .claude/plans/buzzing-twirling-pie.md.
type: project
originSessionId: 89edbad1-7ba1-467f-88c4-5aa53cb86b4b
---
**Incident.** 2026-04-12 07:42 UTC — production PostgREST emitted `column parties.party_id does not exist` (SQLSTATE 42703). Literal bug patched by commit fe18f3e3 on 2026-04-08 (hard DELETE → `soft_delete_party()` RPC) + migration 000231 (RESTRICTIVE `USING(false)` DELETE on `parties`+`items`). The 4-day gap between fix and prod error revealed stale APKs + no 42703 client-side observability.

**Why:** "Turn the water on" (Michael Seibel) — systemic hardening, not another hotfix. 6 phases of defense-in-depth layers, each TDD-locked.

**Plan file:** `.claude/plans/buzzing-twirling-pie.md` (v2, locked).

**7 locked user decisions:**
1. Phase 2 enforcement — **contract-snapshot test** (AST walker → JSON; NOT custom_lint plugin).
2. Phase 3 RPC typed params — **ship now with FULL migration** of ~50 existing `.rpc()` call sites.
3. Phase 1 observability — **Firebase Crashlytics only** (already wired; 0 bundle cost).
4. Phase 4 negative guards — **incident-driven: 2 only** (`parties.party_id` + `billing.firebase_uid`); NOT predictive.
5. Min-version gate — **`parties_delete` only**; items/vendors deferred to their own incidents.
6. Rollout — **Hybrid: P0 → (P1 ∥ P4) → P2 → P3**.
7. Phase 5 (row counts) — **cut**; revive only when silent-no-op surfaces.

**Phase 0 shipped** (commit 951fb198 on feat/tutorial-engine-v5): SchemaDriftReporter interface + FakeSchemaDriftReporter + Riverpod provider + `setSchemaDriftReporterForRetry` setter in postgrest_retry.dart + 4 test files (3 RED + 16 GREEN sentinels) + parties_soft_delete_contract_test.sql + allow-list JSON.

**Phase 1 shipped** (commit 1b52e51d): `isUndefinedColumnError`, `parseUndefinedColumn` (handles two PG message forms + fallback), `reportUndefinedColumn` with MATPU short-circuit, 42703 catch-branch in `withAuthRetry`, and `setAppVersionProviderForRetry` / `setBuildNumberProviderForRetry` hooks. `apps/gst_calculator/lib/services/crashlytics_schema_drift_reporter.dart` forwards tags via `FirebaseCrashlytics.setCustomKey` + `recordError(fatal:false)` with `schema_drift.{kind}.{table}.{column}` reason. `main.dart` wires reporter + PackageInfo-backed app_version providers. All 19 sentinels GREEN.

**Phase 4 shipped** (same commit): 4 advisory pgTAP files promoted to `critical/`:
- `column_naming_guard_test.sql` (renamed from `billing_column_naming_guard_test.sql`) with the 2026-04-12 `hasnt_column('parties', 'party_id')` guard added.
- `schema_drift_validation_test.sql`, `function_column_typo_guard_test.sql`, `soft_delete_schema_test.sql` — promoted unchanged.
- Critical tier count 37 → 41 (+ new `parties_soft_delete_contract_test.sql` from Phase 0).

**Cultural practice:** Fortnightly drift review ritual — `docs/ops/drift_review_log.md`, 30 min, rotating owner, reviews Crashlytics 42703 dashboard + `supabase db diff` + query_shape.snapshot.json delta.

**Phase 2 shipped** (commit 2c03fc5e): `test/contracts/postgrest_query_shape_test.dart` walks `lib/` via analyzer AST, extracts `(table, column, op)` tuples from 12 verbs + `(rpc, paramKeys)` from `.rpc()` calls. Baseline snapshot with 276 tuples committed at `test/contracts/query_shape.snapshot.json`. Accept-drift workflow: `UPDATE_QUERY_SHAPE_SNAPSHOT=1 flutter test ...`. Explicit `analyzer: ^8.0.0` dev_dep added.

**Phase 3.1 shipped** (same commit): hand-written RpcParams for the soft-delete family (ahead of generator fork):
- `lib/core/rpc_params/rpc_params.dart` — `RpcSoftDeletePartyParams(partyId:)` + `RpcDeleteItemSafeParams(itemId:)`.
- 5 call sites migrated from literal map params to typed `.toJson()`: `item_provider.dart`, `item_repository.dart`, `item_summary_notifier.dart`, `inventory_list_notifier.dart`, `party_review_page.dart`.
- 6 RpcParams unit tests (critical tier) with 2026-04-12 regression sentinels (`toJson()` MUST NOT emit bare `party_id` / `item_id`).

**Phase 3.2 COMPLETE — four batches** (commits 4f812e4e, fb7b897b, a87c6429, 9f544c13): ~60 RPC call sites migrated to typed `RpcParams.toJson()`. **Zero untyped `params: {` map literals remain in lib/** (verified: `grep -rn 'params: {' lib/ --include='*.dart'` returns zero non-doc, non-Realtime-auth matches).

- **3.2a** (4f812e4e): stock adjustment (5), party pricing (3), invoice lifecycle (7), payment (3)
- **3.2b** (fb7b897b): onboarding/auth (7), billing prefs + ID config (13)
- **3.2c** (a87c6429): OCR + import (3), dashboard/reports (4), trash (3)
- **3.2d** (9f544c13): invoice wizard ID flow (2), record payment (2), item editor (9 incl. 3× upsert_item_with_stock), item search (2), OCR search/assign (4), party/HSN search (3), adjust_stock previously-missed (2), debug seed (1)

47+ typed classes in `lib/core/rpc_params/rpc_params.dart` — each `@immutable`, const-constructable, with `toJson()` paired 1:1 against `pg_proc` signature. Heaviest: `RpcUpsertItemWithStockParams` (22 params).

**Follow-ups** (tracked but not in critical path):
- supadart generator fork to auto-emit `RpcParams` classes from `pg_proc` — eliminates hand-written maintenance.
- **Deferred Phase 0 items** — `parties_delete_min_version_test.dart` (needs Remote Config), `integration_test/parties/delete_flow_test.dart` (needs emulator + `--flavor dashboard`), RPC manifest test (pairs with generator fork).

**Audits:** design-postgres-tables (schema layer), brainstorming (systems + YAGNI), test-integration-principles (TDD + sentinels). All 3 surfaced complementary gaps → plan v2 resolved them before implementation.
