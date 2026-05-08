# Active Memory Kernel - billing_flutter_plugins

This file is the active decision kernel. Detailed `feedback_*.md` and `project_*.md` files are evidence archives, not automatically active doctrine. If this kernel conflicts with an archive, this kernel wins unless the user explicitly reopens that archived decision.

Keep this file compact. Move subsystem detail, incident narratives, and completed project snapshots into their specific memory files.

## Labels

- `P0 HARD` - must affect behavior immediately; violation can cause prod, user, revenue, security, or data-integrity damage.
- `P1 DEFAULT` - preferred doctrine; override only with a stated trade-off.
- `P2 CONTEXTUAL` - read when touching that subsystem.
- `P3 ARCHIVE` - historical evidence only; do not generalize into new plans.
- `EVIDENCE-GATED` - change only with telemetry, probe output, incident evidence, or verified shipped-code facts.

## Conflict Resolver

1. Current explicit user instruction wins.
2. This active kernel wins over archived memory files.
3. User-locked decisions win over audit-agent suggestions.
4. Verified repository facts at HEAD win over plan text and assumptions.
5. Safety bans and destructive-command rules win over convenience.
6. Archived project memories are context for that subsystem only; never generalize them into default behavior.
7. If a newer archive contradicts this kernel and the answer changes implementation shape, ask the user before applying it.

## User-Locked Doctrine

- P0 HARD - Do not add runtime flags, kill switches, remote-config branches, or client toggles for UI/UX changes unless the user explicitly asks for one. This includes visual styling, dashboard IA, navigation shell, forms, banners, and paywall screens. UI is client-side: ship the direct change, review it, test it, and keep the diff reversible.
- P0 HARD - Zero-consumer source code is deleted in the same PR. No `30d`, `60d`, or `90d` sunset clocks for orphan enums, getters, fields, abstract classes, deprecated impls, placeholders, or future-use code. Details: `feedback_no_consumer_delete_now.md`.
- P0 HARD - Deprecation exists only for real current callers, persisted data, external consumers, or explicit user requirement. Zero callers means delete, not deprecate.
- P0 HARD - YC/Seibel "turn on the water" doctrine is split by reversibility: PREDICT and guard irreversible pipes (schema migrations, ACL grants, RPC contract changes, KV cohort key shapes, ledger writes) before deploy with pgTAP, dry-run, ledger inspection, scope gates; only RUN water-test exposure on reversible runtime behavior (Workers Logs distributions, error rates, latency, cache hit/miss patterns) with bucketed structured-log evidence and named rollback thresholds. Details: `feedback_predict_irreversible_test_reversible.md`. Supersedes the older "smallest reversible change exposes the weak pipe" framing — that framing was uniform; the actual rule splits.
- P1 DEFAULT - Prefer deletion, simplification, and direct implementation over preserving optionality. Git history is the archive for removed source-level code.

## Workflow And Git

- P0 HARD - Never create surprise branches. Work on the loaded branch; for isolation use worktrees. Details: `feedback_use_worktrees_not_branches.md`.
- P0 HARD - Push immediately after every commit. If push is denied, escalate via AskUserQuestion; never silently leave commits unpushed. Details: `feedback_push_after_every_commit.md`.
- P1 DEFAULT - Const-context equivalence exception: if a `static const` initializer blocks a token reference, a verified equivalent literal may be used only with a comment capturing equivalence, reason, and refactor trigger. Details: `feedback_const_context_equivalence_exception.md`.
- P0 HARD - Never run destructive Supabase rollback/reset commands against linked/prod databases. Use forward-fix migrations. Details: `feedback_never_supabase_migration_down_linked.md`.
- P1 DEFAULT - P0 incidents ship the minimum viable fix plus minimum proof. ADR rewrites, rule CSVs, manifests, broad observability, and memory cleanup move to follow-up. Details: `feedback_p0_fix_minimum_viable_defer_doctrine.md`.
- P1 DEFAULT - Before building on "already shipped" claims, verify HEAD with file/symbol/git evidence. Details: `feedback_shipped_claim_audit_works.md`.
- P0 HARD - Subsystem authoring verifies HEAD facts before SQL/migration/Worker code: table existence (grep CREATE TABLE), KV key prefixes (read Worker `cache-keys.ts`, not CLAUDE.md), RPC parameter counts + canonical body location across forward fixes (grep `<rpc_name>` migrations, sort -V, tail -3), exact filter predicates at line numbers (open migration, read WHERE verbatim), Vault auth pattern (`public.get_secret`, not `current_setting`), skill-mirror dual root (.claude/skills + .agents/skills both edited or symlinked). Reason: 2026-05-03 plan rejection due to six HEAD-fact mistakes that would have caused production incidents. Details: `feedback_subsystem_authoring_verifies_head_facts.md`.

## Planning And Audits

- P0 HARD - Audits expose risks; they do not renegotiate locked user commitments. Preserve the commitment and carve out the audit concern explicitly. Details: `feedback_dont_let_audits_override_user_commitments.md`.
- P1 DEFAULT - For destructive drops, contract changes, or retirements, produce a file:line impact map across Flutter, Worker/Edge, SQL, tests, and migrations before finalizing. Details: `feedback_three_layer_impact_map_for_audits.md`.
- P1 DEFAULT - Contract-first file:line plans are mandatory for schema, RPC/API, handler, migration, auth, billing, payment, or cross-layer changes. For small localized UI/source edits, use a short plan and direct verification; do not inflate into pseudocode-level planning.
- P1 DEFAULT - After audit corrections, rewrite the canonical plan inline. Do not append contradictory "Audit Amendments" sections. Details: `feedback_canonical_single_sequence_plans.md`.
- P1 DEFAULT - If the user provides a structured audit with locked decisions/directives, adopt it rather than re-interviewing. Details: `feedback_user_led_parallel_audits.md`.
- P1 DEFAULT - Keep AskUserQuestion interviews focused: 2-3 rounds, 8-12 high-leverage questions unless the user explicitly asks for more. Details: `feedback_interview_pacing.md`.
- P0 HARD - Before relocating, renaming, or adding rule IDs in skill CSVs, grep the canonical CSV files at HEAD. Don't propose IDs that already exist or count files from earlier-session synthesis. Don't propose new pump/time/lifecycle/state-discipline rules without first checking `FLAKY-*` / `TASTE-*` / `CHAIN-*` for existing coverage. Rule-reference linters and coverage checkers are distinct concerns — don't conflate. Details: `feedback_verify_rule_ids_at_head.md`.

## Rollout And Observability

- P0 HARD - Source-level dead code is not a production defense layer. Do not apply telemetry windows to unreachable source or UI branches.
- P1 EVIDENCE-GATED - Production defense layers with real traffic must earn their cost by catching a distinct failure mode. If a reachable defense layer uniquely catches zero measured cases for 30 days, it is a deletion candidate. Details: `feedback_defence_in_depth_budget.md`.
- P1 EVIDENCE-GATED - Shipping or reverting production hardening requires telemetry, probe, incident, or shipped-code evidence. This gate does not block deleting dead code or shipping small direct UI/source changes. Details: `feedback_evidence_gates_apply_recursively.md`.
- P1 DEFAULT - Risky data-plane cutovers need the smallest useful evidence channel before cutover: structured logs or counters first; health endpoints, synthetic probes, and dashboards only when blast radius warrants them. Details: `feedback_observability_before_cutover.md`.
- P0 HARD - Production deploy smokes whose correctness is required for the rollout MUST fail closed when prerequisites are missing — never soft-skip. Defence-in-depth probes may soft-skip; load-bearing rollout proofs cannot. Details: `feedback_production_smoke_fails_closed.md`.
- P1 DEFAULT - Water-tests need bucketed evidence, usually structured logs with explicit reason fields. Durable tables and dashboards are required only when multi-day aggregation is needed. Details: `feedback_seibel_water_test_requires_bucketing.md`.
- P1 DEFAULT - Prefer structured Workers Logs over speculative Analytics Engine or unused telemetry bindings. Details: `feedback_structured_logs_over_analytics_engine.md`.

## Database, Security, And Secrets

- P0 HARD - Prod is permanent, local is disposable. Local-Docker recovery plans must not bundle prod-touching steps. Every linked-mutating command needs an explicit user gate even if earlier local steps were approved. Asymmetric lifespans → asymmetric risk appetite. Details: `feedback_prod_permanent_local_disposable.md`.
- P0 HARD - Never edit deployed migrations. Add a new forward migration. Details: `feedback_never_edit_migrations.md`.
- P0 HARD - Before editing or proposing edits to any existing Supabase migration, run `supabase migration list --linked` and inspect the Remote column. If the version is listed remotely, restore/leave the file immutable and create a new forward migration with `make migration-new NAME=...`.
- P0 HARD - `supabase db push --linked --dry-run` validates parsing only. Dependency failures surface during apply; iterate carefully and document SQLSTATE fixes. Details: `feedback_iterate_db_push_linked_pattern.md`.
- P0 HARD - Migration scope gate before any linked db push: enumerate the exact pending set (`cat LATEST_MIGRATION` + `supabase migration list --linked`) and confirm the push targets only the named migration. `supabase db push --linked` applies ALL pending migrations; if unrelated work is pending, STOP and surface to user. Details: `feedback_migration_scope_gate_before_push.md`.
- P0 HARD - Before `DROP COLUMN`, enumerate and explicitly handle constraints, triggers, RPCs, views, indexes, FKs, and generated expressions. Details: `feedback_drop_column_cascade_audit.md`.
- P0 HARD - No anon-callable Supabase RPCs for user sync. Use the approved server-side path. Details: `feedback_no_anon_supabase_rpc.md`.
- P0 HARD - Service role keys are deprecated for new code. Do not introduce new service-role usage without explicit user approval and documented containment. Details: `feedback_service_role_key_deprecated.md`.
- P0 HARD - Cloudflare Dashboard is the source of truth for secrets. Deploy scripts must not bulk-put/delete dashboard secrets. Details: `feedback_dashboard_is_source_of_truth_strict.md`.
- P0 HARD - Chaos/canary experiments never run on Razorpay, subscription, GST, webhook, or pg_net billing paths. Details: `feedback_chaos_never_on_billing_paths.md`.
- P0 HARD - Compact 42501 Auth/RPC doctrine: Firebase Admin `setCustomUserClaims` replaces the whole custom-claims object, so fetch/merge existing `customClaims` before setting `role: "authenticated"`; PostgREST `user_name: authenticator` is only the connection role, while the effective request role comes from JWT role switching and missing/no-role Firebase JWTs route to Supabase `anon`; ACL hardening does not recover stale clients, only claim repair plus token refresh/re-auth/token expiry does; SECDEF RPC ACLs use CLOSED for financial/write/state-mutation RPCs (revoke PUBLIC/anon/authenticator, grant authenticated/service_role only) and SWEEP only for explicitly body-guarded user-facing RPCs (early null-auth guard, canonical bare `RAISE EXCEPTION 'Not authenticated';` / P0001, anon grant so clean auth UX survives role-claim races); helper-based identity extraction through `firebase_uid()`, `has_billing_access`, `user_billing_id`, or JWT-claim reads is a SWEEP candidate only for non-financial user-facing RPCs, while financial/write/state-mutation RPCs stay CLOSED unless a human approves an exception with idempotency, duplicate-submit, and lock/allocation proof; never grant client RPC execution to `authenticator`; Payment/Razorpay/subscription write RPCs must not enter 42501 retry without per-RPC idempotency proof; `withAuthRetry` already force-refreshes through existing `SupabaseRestClient.refreshAuth(forceRefresh: true)` / client recreation paths, so do not add a third raw `getIdToken(true)` strategy inside it; missing-role repair UX is explicit repair-then-retry (user taps Retry, callable repairs own role claim, client force-refreshes Firebase/Supabase auth, original RPC reissues); PGRST* values are PostgREST `error.code` values, while 42501/P0001 are PostgreSQL SQLSTATEs; client `[AUTH_RETRY]` logs must not contain raw UID/sub or token/JWT material, though server Functions logs may include explicit UID fields. Details: `feedback_postgrest_sweep_rpc_classification.md`, `feedback_sweep_guard_p0001_mapper_parity.md`, `feedback_42501_retry_policy_call_site_opt_in.md`, ADR-023, ADR-024.
- P1 DEFAULT - Firebase Auth V1 `onCreate` triggers do not repair claims on sign-in. Per-sign-in blocking repair requires Identity Platform; existing-user repair runs through admin/support tools.
- P1 DEFAULT - Canonical Supabase migration sentinel is `supabase/migrations/LATEST_MIGRATION` (root copy is stale). Read before proposing a number; create new files via `make migration-new NAME=...`. Details: `reference_supabase_migrations_latest_sentinel.md`.
- P1 DEFAULT - Token refresh paths must call `SupabaseRestClient.instance.refreshAuth()` (singleton). `supabaseRestProvider` and `postgrestProvider` throw `StateError` on null JWT (Layer 1.5 transport guard at `supabase_provider.dart:142-159`). Details: `feedback_singleton_for_token_refresh_not_provider.md`.

## UI And Product Surfaces

- P0 HARD - No UI runtime flags unless the user explicitly asks. Old banner memories that say to preserve UI paths behind flags are superseded by this rule.
- P1 DEFAULT - UI/UX changes ship as small direct diffs with visual review, widget/golden tests where useful, and fast git/hotfix rollback. Do not keep legacy UI branches for optionality.
- P1 DEFAULT - System-back / pop handlers in surfaces with debounced input must check raw controller state in addition to (or instead of) the debounced derivative; otherwise typing-then-back before debounce settles can pop the route instead of clearing input. Add a FakeAsync widget test for this race. Details: `feedback_predebounce_ui_back_handler.md`.
- P1 DEFAULT - Surface-designer hard gates require a `*.surface.md` artifact at the canonical handoff location; embedded JSON inside a planning doc does not satisfy the contract. Same applies to wizard-designer, pdf-template-design, paywall-designer. Details: `feedback_surface_designer_artifact_handoff.md`.
- P1 DEFAULT - Dart privacy is library-scoped. A leading-underscore widget (`_Foo`) cannot be imported from another file even with explicit `import`; declare it in the same file as its consumer, use `part`/`part of`, or make it public. Details: `feedback_dart_privacy_library_scoped.md`.
- P1 DEFAULT - When a fixed bottom bar overlays a scrollable body, reserve the inset in EXACTLY one place — an `AnimatedPadding` around the body switcher — never also add bottom padding to the scrollable content (grid, list, results). Doubles space and breaks on collapse. Details: `feedback_single_source_bottom_inset.md`.
- P1 DEFAULT - Dashboard ticker and summary-card leaves stay presentation-only; page/container widgets own provider reads. Details: `feedback_presentation_only_leaf_widgets.md`.
- P1 DEFAULT - If UI work changes data source, count semantics, or routing, call it information architecture and add source/routing tests. This classification does not imply a runtime flag. Details: `feedback_ia_refactor_not_pure_visual.md`.
- P1 DEFAULT - Visible dashboard tickers use existing aggregation RPC/MV-backed providers; do not derive counts from rendered lists unless the user accepts drift risk. Details: `feedback_dashboard_tickers_use_mv_backed_rpcs.md`.
- P1 DEFAULT - One-time paywall/upsell gates must fail-open (try/catch around launch + markShown + analytics) and mark the SharedPrefs "shown" flag ONLY when the wizard actually rendered. Coordinator preflight short-circuits (null UID, paid bypass) emit `result: 'bypassed'` with explicit `via:` discriminator and never mark shown. Details: `feedback_paywall_gate_fail_open_and_render_proof.md`.
- P2 CONTEXTUAL - Banner-specific project files are historical/contextual unless actively debugging banner code. Do not copy their flag-heavy rollout posture into new UI work.

## Testing And Tooling Gotchas

- P1 DEFAULT - Flutter widget tests use the standard 390x844 portrait viewport helper unless a test intentionally needs a different size.
- P1 DEFAULT - pgTAP files do not use SAVEPOINT/ROLLBACK TO SAVEPOINT; use role switching and pgTAP exception helpers. Details: `feedback_pgtap_savepoint_forbidden.md`.
- P1 DEFAULT - pgTAP behavioral assertions outrank introspection: `pg_proc.prosrc ILIKE '%pattern%'` is a secondary guard, not a correctness gate; `RAISE NOTICE` log content cannot be reliably asserted in pgTAP. Pair every prosrc check with a behavioral test; verify NOTICE/log output manually post-deploy. Details: `feedback_pgtap_behavioral_over_introspection.md`.
- P1 DEFAULT - Riverpod provider-chain tests use `ProviderContainer`, dispose containers, and override providers correctly.
- P1 DEFAULT - Prefer Dart LSP for symbol navigation and Grep/Glob for text, string, comment, and file-pattern search.
- P2 CONTEXTUAL - Read subsystem memory files only when touching that subsystem; do not preload every gotcha into active planning.
- P1 DEFAULT - `withAuthRetry` callers must read `postgrestProvider` INSIDE the retry closure. Capturing the client outside breaks Strategy 2 client recreation (force-recreate disposes the old PostgrestClient). Details: `feedback_withauthretry_read_provider_inside_closure.md`.
- P1 DEFAULT - 42501 retry policy is call-site opt-in via `retryablePermissionDeniedFunctions` parameter on `withAuthRetry`. No global blanket-retry of "permission denied for function". Details: `feedback_42501_retry_policy_call_site_opt_in.md`.
- P1 DEFAULT - `withAuthRetry` already force-refreshes through `SupabaseRestClient.refreshAuth(forceRefresh: true)` / client recreation. Do not add a third raw `getIdToken(true)` strategy inside `withAuthRetry`; do call-site role-claim verification after exhaustion where needed.
- P1 DEFAULT - Firebase/GA4 analytics: `AnalyticsService.logEvent` signature is `Future<void> logEvent(String, Map<String, Object>)` (NOT `Object?`); only String/int/double param values supported — encode booleans as `1/0` ints. Wrap async calls in `unawaited(...catchError)` (synchronous try/catch can't catch the unawaited Future). Tests use a `CapturingAnalyticsService implements AnalyticsService`, not `NoOpAnalyticsService`, when asserting events. Privacy: `debug_log_collector.dart` persists logs to disk — never log raw user query text; log `query_length` only. Details: `feedback_analytics_int_booleans_capturing_test.md`.
- P1 DEFAULT - For Riverpod feature tests where widget → notifier → external dependency, assert via the dependency seam (override the data source provider with a recording fake), NOT by overriding the notifier with a spy. Survives notifier refactors and respects route-local `ProviderScope` overrides. Details: `feedback_data_source_seam_beats_notifier_spy.md`.
- P1 DEFAULT - Tests touching `PaywallCoordinator.showPaywall` must override `userIdProvider` (non-null), `userContextProvider`, `subscriptionProvider`, `postAuthTeaserProvider`, `paywallSessionTrackerProvider`, `backendReadyProvider`, AND `isIndianUserProvider` (or fake `loadIndianBridgeFn`). Without these, coordinator preflight masks behavior or Indian bridge loader flakes. Reset `PaywallCoordinator.resetForTesting()` + restore `debugPrint` in BOTH `setUp` and `tearDown`. Prefer coordinator/target-handler-level regression over full reel-video UI for CTA-chain coverage. Details: `feedback_paywall_coordinator_test_overrides.md`.
- P1 DEFAULT - `test/manifest.json` deltas need arithmetic: module `testCount = sum(target.testCount)`; `path` points at the SOURCE dir (not test dir); update both `summary.totalModules` AND `summary.testedModules`; recompute `coveragePercent`. Run `make check-rule-refs` + `make check-widget-surfaces` after edits. Details: `feedback_manifest_delta_arithmetic.md`.
- P1 DEFAULT - Sample-presence deploy smokes accumulate across the loop and assert `positives > 0` once after; never per-row fail when individual rows can legitimately lack the sample. Details: `feedback_smoke_sample_presence_accumulator.md`.
- P1 DEFAULT - JSONB sibling columns of strictly-typed payloads (e.g. `preview_cta_config` next to `cta_config`) decode as `Map<String, dynamic>?` at the carrier model — convert to strict typed shape only at the consumer. Strict at the carrier silently loses legitimate shape variants. Details: `feedback_jsonb_decode_tolerant_map.md`.
- P1 DEFAULT - Riverpod 3.2.1 `overrideWith` signature varies by provider type: functional `Provider/FutureProvider/StreamProvider` keep `(ref) =>`; class-based generated `(Async)NotifierProvider` use zero-arg `() =>`. Don't mass-sweep; respect the provider type. Details: `feedback_overridewith_signature_per_provider_type.md`.
- P0 HARD - Don't invalidate `authProvider` before Firebase signOut. `AuthNotifier.build()` reads `FirebaseAuth.instance.currentUser` synchronously; pre-Firebase invalidation rebuilds with the same signed-in user. Cross-tab logout invalidates `billingContextProvider` ONLY; FirebaseAuth stream drives `authProvider` reset. Details: `feedback_authprovider_no_invalidate_pre_firebase_signout.md`.
- P1 DEFAULT - Cold-start interleaving symptoms are usually a real `ref.read` reactivity bug on a bootstrap provider, not flakiness. Convert to `ref.watch` or `ref.listen + invalidateSelf`; do NOT reach for `Clock.fixed()` injection as a workaround — it masks the architectural bug. Details: `feedback_coldstart_real_dep_bug_not_flaky.md`.
- P1 DEFAULT - Don't speculatively edit design tokens (`BillingSpacing`, color tokens, etc.) on visual-sentinel test failures. Require an exact failing assertion at file:line; intentional asymmetries in token values can be misread as regressions. Defer to ledger entry until specific assertion surfaces. Details: `feedback_no_visual_sentinel_speculation.md`.
- [tutorials_admin Round-3 status](project_tutorials_admin_seamless_round3_status.md) — 9 of 10 commits shipped 2026-04-26; deferred items + invariants pinned by tests
- [Tutorials two placement systems](project_tutorials_two_placement_systems.md) — `banner_placement_config` (serving) vs `tutorial_sources` (graph traversal); both must be updated for placement changes
- [Spec claims must match client capability](feedback_spec_claims_must_match_client_capability.md) — before claiming Worker reads field X, grep client code that X is sent
- [Banner v5 corpus is permissive](feedback_v5_corpus_permissive_doctrine.md) — banner predicates use `is_media_ready=TRUE`, NOT `is_reels_eligible`; pinned by `banner_v5_corpus_stays_permissive_test.sql`
- [Worker can't see hidden v5 columns](feedback_worker_v5_returns_shape_constraints.md) — v5 RETURNS 15 cols projecting tutorials_reels; banner_id/salience/sentinel are SQL-only; rotation lives in SQL ORDER BY + Worker positional shuffle
- [Two-pipe discount accounting](project_payment_two_pipe_discount.md) — global vs ad-hoc split; 000432 dropped F1; balance-trigger gap CLOSED by 000442 (not 000434; that file shipped banner views)
- [Invoice allocation triggers](project_invoice_allocation_trigger_facts.md) — trg_fn_validate_invoice_allocation is BEFORE INSERT; chk_payment_allocations_allocation_valid rejects (0,0); RPC ORDER BY invoice_id ASC
- [Invoice ID allocator reclaim](project_invoice_id_allocator_reclaim.md) — get_next_available_invoice_id reclaims same-FY tombstones first; collision scope (firm,mode,FY,number); prefix is informational
- [upsert_party SWEEP 000457](project_upsert_party_sweep_000457.md) — `upsert_party` is SWEEP as of 000457; only `save_payment_with_allocations` v1/v2 remain CLOSED from the 000456 set.
- [pgTAP lock-order via pg_proc.prosrc](feedback_pgtap_lock_order_structural.md) — single-session pgTAP can't prove concurrent lock order; assert ORDER BY pattern in source instead
- P0 HARD - Payment allocation must not hard-clamp to outstanding balance. Preserve "Over by ₹X" UX, disabled-save, and analytics-event contracts. Details: [`feedback_payment_allocation_no_hard_clamp.md`](feedback_payment_allocation_no_hard_clamp.md).
- P0 HARD - Tutorials APIs stay REST/RPC + KV; do not migrate to GraphQL. Future GraphQL surface is admin-only, read-only, persisted-query-only, bearer-gated, parity-proven, sunset-bounded. Details: [`feedback_graphql_rejected_for_tutorials.md`](feedback_graphql_rejected_for_tutorials.md) and ADR-019.