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
- P0 HARD - YC/Seibel "turn on the water" means ship the smallest reversible change that can expose the weak pipe, with targeted leak detection. It does not mean speculative toggles, broad fallback trees, or prolonged bake windows.
- P1 DEFAULT - Prefer deletion, simplification, and direct implementation over preserving optionality. Git history is the archive for removed source-level code.

## Workflow And Git

- P0 HARD - Never create surprise branches. Work on the loaded branch; for isolation use worktrees. Details: `feedback_use_worktrees_not_branches.md`.
- P0 HARD - Push immediately after every commit. If push is denied, escalate via AskUserQuestion; never silently leave commits unpushed. Details: `feedback_push_after_every_commit.md`.
- P0 HARD - Never run destructive Supabase rollback/reset commands against linked/prod databases. Use forward-fix migrations. Details: `feedback_never_supabase_migration_down_linked.md`.
- P1 DEFAULT - P0 incidents ship the minimum viable fix plus minimum proof. ADR rewrites, rule CSVs, manifests, broad observability, and memory cleanup move to follow-up. Details: `feedback_p0_fix_minimum_viable_defer_doctrine.md`.
- P1 DEFAULT - Before building on "already shipped" claims, verify HEAD with file/symbol/git evidence. Details: `feedback_shipped_claim_audit_works.md`.

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
- P1 DEFAULT - Water-tests need bucketed evidence, usually structured logs with explicit reason fields. Durable tables and dashboards are required only when multi-day aggregation is needed. Details: `feedback_seibel_water_test_requires_bucketing.md`.
- P1 DEFAULT - Prefer structured Workers Logs over speculative Analytics Engine or unused telemetry bindings. Details: `feedback_structured_logs_over_analytics_engine.md`.

## Database, Security, And Secrets

- P0 HARD - Prod is permanent, local is disposable. Local-Docker recovery plans must not bundle prod-touching steps. Every linked-mutating command needs an explicit user gate even if earlier local steps were approved. Asymmetric lifespans → asymmetric risk appetite. Details: `feedback_prod_permanent_local_disposable.md`.
- P0 HARD - Never edit deployed migrations. Add a new forward migration. Details: `feedback_never_edit_migrations.md`.
- P0 HARD - `supabase db push --linked --dry-run` validates parsing only. Dependency failures surface during apply; iterate carefully and document SQLSTATE fixes. Details: `feedback_iterate_db_push_linked_pattern.md`.
- P0 HARD - Before `DROP COLUMN`, enumerate and explicitly handle constraints, triggers, RPCs, views, indexes, FKs, and generated expressions. Details: `feedback_drop_column_cascade_audit.md`.
- P0 HARD - No anon-callable Supabase RPCs for user sync. Use the approved server-side path. Details: `feedback_no_anon_supabase_rpc.md`.
- P0 HARD - Service role keys are deprecated for new code. Do not introduce new service-role usage without explicit user approval and documented containment. Details: `feedback_service_role_key_deprecated.md`.
- P0 HARD - Cloudflare Dashboard is the source of truth for secrets. Deploy scripts must not bulk-put/delete dashboard secrets. Details: `feedback_dashboard_is_source_of_truth_strict.md`.
- P0 HARD - Chaos/canary experiments never run on Razorpay, subscription, GST, webhook, or pg_net billing paths. Details: `feedback_chaos_never_on_billing_paths.md`.
- P1 DEFAULT - Canonical Supabase migration sentinel is `supabase/migrations/LATEST_MIGRATION` (root copy is stale). Read before proposing a number; create new files via `make migration-new NAME=...`. Details: `reference_supabase_migrations_latest_sentinel.md`.
- P1 DEFAULT - Token refresh paths must call `SupabaseRestClient.instance.refreshAuth()` (singleton). `supabaseRestProvider` and `postgrestProvider` throw `StateError` on null JWT (Layer 1.5 transport guard at `supabase_provider.dart:142-159`). Details: `feedback_singleton_for_token_refresh_not_provider.md`.

## UI And Product Surfaces

- P0 HARD - No UI runtime flags unless the user explicitly asks. Old banner memories that say to preserve UI paths behind flags are superseded by this rule.
- P1 DEFAULT - UI/UX changes ship as small direct diffs with visual review, widget/golden tests where useful, and fast git/hotfix rollback. Do not keep legacy UI branches for optionality.
- P1 DEFAULT - Dashboard ticker and summary-card leaves stay presentation-only; page/container widgets own provider reads. Details: `feedback_presentation_only_leaf_widgets.md`.
- P1 DEFAULT - If UI work changes data source, count semantics, or routing, call it information architecture and add source/routing tests. This classification does not imply a runtime flag. Details: `feedback_ia_refactor_not_pure_visual.md`.
- P1 DEFAULT - Visible dashboard tickers use existing aggregation RPC/MV-backed providers; do not derive counts from rendered lists unless the user accepts drift risk. Details: `feedback_dashboard_tickers_use_mv_backed_rpcs.md`.
- P2 CONTEXTUAL - Banner-specific project files are historical/contextual unless actively debugging banner code. Do not copy their flag-heavy rollout posture into new UI work.

## Testing And Tooling Gotchas

- P1 DEFAULT - Flutter widget tests use the standard 390x844 portrait viewport helper unless a test intentionally needs a different size.
- P1 DEFAULT - pgTAP files do not use SAVEPOINT/ROLLBACK TO SAVEPOINT; use role switching and pgTAP exception helpers. Details: `feedback_pgtap_savepoint_forbidden.md`.
- P1 DEFAULT - Riverpod provider-chain tests use `ProviderContainer`, dispose containers, and override providers correctly.
- P1 DEFAULT - Prefer Dart LSP for symbol navigation and Grep/Glob for text, string, comment, and file-pattern search.
- P2 CONTEXTUAL - Read subsystem memory files only when touching that subsystem; do not preload every gotcha into active planning.
- P1 DEFAULT - `withAuthRetry` callers must read `postgrestProvider` INSIDE the retry closure. Capturing the client outside breaks Strategy 2 client recreation (force-recreate disposes the old PostgrestClient). Details: `feedback_withauthretry_read_provider_inside_closure.md`.
- P1 DEFAULT - 42501 retry policy is call-site opt-in via `retryablePermissionDeniedFunctions` parameter on `withAuthRetry`. No global blanket-retry of "permission denied for function". Details: `feedback_42501_retry_policy_call_site_opt_in.md`.
- [tutorials_admin Round-3 status](project_tutorials_admin_seamless_round3_status.md) — 9 of 10 commits shipped 2026-04-26; deferred items + invariants pinned by tests
- [Two-pipe discount accounting](project_payment_two_pipe_discount.md) — global vs ad-hoc split; 000432 dropped F1; balance-trigger gap closed by queued 000434
- [Invoice allocation triggers](project_invoice_allocation_trigger_facts.md) — trg_fn_validate_invoice_allocation is BEFORE INSERT; chk_payment_allocations_allocation_valid rejects (0,0); RPC ORDER BY invoice_id ASC
- [pgTAP lock-order via pg_proc.prosrc](feedback_pgtap_lock_order_structural.md) — single-session pgTAP can't prove concurrent lock order; assert ORDER BY pattern in source instead
