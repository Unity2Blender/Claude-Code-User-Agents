# Active Memory Kernel - billing_flutter_plugins

This file is the active decision kernel. Detailed `feedback_*.md` and `project_*.md` files are evidence archives, not automatically active doctrine. If this kernel conflicts with an archive, this kernel wins unless the user explicitly reopens that archived decision. <!-- LOAD-BEARING -->

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

- P0 HARD - Do not add runtime flags, kill switches, remote-config branches, or client toggles for UI/UX changes unless the user explicitly asks for one. This includes visual styling, dashboard IA, navigation shell, forms, banners, and paywall screens. UI is client-side: ship the direct change, review it, test it, and keep the diff reversible. <!-- LOAD-BEARING -->
- P0 HARD - Zero-consumer source code is deleted in the same PR. No `30d`, `60d`, or `90d` sunset clocks for orphan enums, getters, fields, abstract classes, deprecated impls, placeholders, or future-use code. Details: `feedback_no_consumer_delete_now.md`. <!-- LOAD-BEARING -->
- P0 HARD - Deprecation exists only for real current callers, persisted data, external consumers, or explicit user requirement. Zero callers means delete, not deprecate.
- P0 HARD - YC/Seibel "turn on the water" doctrine is split by reversibility: PREDICT and guard irreversible pipes (schema migrations, ACL grants, RPC contract changes, KV cohort key shapes, ledger writes) before deploy with pgTAP, dry-run, ledger inspection, scope gates; only RUN water-test exposure on reversible runtime behavior (Workers Logs distributions, error rates, latency, cache hit/miss patterns) with bucketed structured-log evidence and named rollback thresholds. Details: `feedback_predict_irreversible_test_reversible.md`. Supersedes the older "smallest reversible change exposes the weak pipe" framing — that framing was uniform; the actual rule splits. <!-- LOAD-BEARING -->
- P1 DEFAULT - Prefer deletion, simplification, and direct implementation over preserving optionality. Git history is the archive for removed source-level code.

## Workflow And Git

- P0 HARD - Never create surprise branches. Work on the loaded branch; for isolation use worktrees. Details: `feedback_use_worktrees_not_branches.md`.
- P0 HARD - Push immediately after every commit. If push is denied, escalate via AskUserQuestion; never silently leave commits unpushed. Details: `feedback_push_after_every_commit.md`.
- P0 HARD - Never add `Co-Authored-By: Claude` (or any Anthropic-branded attribution) to commits unless user-confirmed the CLI is Claude Code. OpenCode/Codex/other may run different models. Default: omit and ask. Details: `feedback_no_false_co_author_attribution.md`. <!-- LOAD-BEARING -->
- P1 DEFAULT - Const-context equivalence exception: if a `static const` initializer blocks a token reference, a verified equivalent literal may be used only with a comment capturing equivalence, reason, and refactor trigger. Details: `feedback_const_context_equivalence_exception.md`. <!-- LOAD-BEARING -->
- P0 HARD - Never run destructive Supabase rollback/reset commands against linked/prod databases. Use forward-fix migrations. Details: `feedback_never_supabase_migration_down_linked.md`.
- P1 DEFAULT - P0 incidents ship the minimum viable fix plus minimum proof. ADR rewrites, rule CSVs, manifests, broad observability, and memory cleanup move to follow-up. Details: `feedback_p0_fix_minimum_viable_defer_doctrine.md`. <!-- LOAD-BEARING -->
- P1 DEFAULT - Before building on "already shipped" claims, verify HEAD with file/symbol/git evidence. Details: `feedback_shipped_claim_audit_works.md`.
- P0 HARD - Subsystem authoring verifies HEAD facts before SQL/migration/Worker code: table existence (grep CREATE TABLE), KV key prefixes (read Worker `cache-keys.ts`, not CLAUDE.md), RPC parameter counts + canonical body location across forward fixes (grep `<rpc_name>` migrations, sort -V, tail -3), exact filter predicates at line numbers (open migration, read WHERE verbatim), Vault auth pattern (`public.get_secret`, not `current_setting`), skill-mirror dual root (.claude/skills + .agents/skills both edited or symlinked), provider/symbol existence at HEAD before promising test-cell counts (grep -rln before promising N), CI script existence before claiming "create" vs "extend" (ls .claude/scripts/ first), CI tool availability (`command -v <tool>` + `brew/apt install` fallback before naming as gate dependency). Reason: 2026-05-03 plan rejection due to six HEAD-fact mistakes + 2026-05-09 PR1.6 cycle three additional misses. Details: `feedback_subsystem_authoring_verifies_head_facts.md`. <!-- LOAD-BEARING -->
- P0 HARD - Agent column-list summaries are paraphrased, even with file:line citations. For SQL plans naming columns/projections/PKs/dropped-columns: Read the CREATE TABLE block verbatim at the cited line offset; do not trust Explore/general-purpose agent column reports as authoritative. Reason: 2026-05-14 dump-RPC plan rejected for 5 wrong column names (placement_id+is_sentinel dropped by 000411; banner_placement_config.name/is_active don't exist; module_id vs module_key; target_key vs key; reason vs reason_code+comparison_op) — agents had cited the right files but summarized columns. Details: `feedback_agent_column_names_not_authoritative.md`. <!-- LOAD-BEARING -->
- P0 HARD - Before authoring any validator-strict artifact (`*.surface.md`, `test/manifest.json`, schema configs), READ the validator source AND a working example FIRST. Cite only rule IDs that exist in `.claude/skills/<skill>/index/rules.json`. Details: `feedback_read_validator_source_before_authoring_artifacts.md`. <!-- LOAD-BEARING -->
- P0 HARD - Re-run `cat supabase/migrations/LATEST_MIGRATION` IMMEDIATELY before plan close and before `make migration-new`; do not trust explore-phase sentinel snapshots. Migration ledger churns within hours, faster than other HEAD facts. Reason: 2026-05-13 Reddit-audit plan rejection — explore reported `000503` latest, but `000504_npt_reel_video_url_repair` shipped between explore and plan-close; the plan named the new audit-fix migration `000504_*` and collided. Details: `feedback_migration_ledger_churns_recheck_at_close.md`. <!-- LOAD-BEARING -->
- P0 HARD - Re-run `git rev-parse HEAD` at every plan revision; never trust SHAs from earlier revisions. Details: `feedback_v12_head_pin_staleness.md`.
- P0 HARD - Supabase advisor flags (`mcp__claude_ai_Supabase__get_advisors` or local Supabase Linter) are real production-impacting findings — never noise. Run pre-flight before migration plan close + post-flight to verify count decrease. New tables MUST `ALTER TABLE ... ENABLE ROW LEVEL SECURITY` + `CREATE POLICY service_role_full FOR ALL TO service_role USING (TRUE) WITH CHECK (TRUE)` in the same migration. ACL REVOKE alone does NOT clear `rls_disabled_in_public`. Details: `feedback_supabase_advisor_flags_are_findings.md`. <!-- LOAD-BEARING -->

## Planning And Audits

- P0 HARD - Audits expose risks; they do not renegotiate locked user commitments. Preserve the commitment and carve out the audit concern explicitly. Details: `feedback_dont_let_audits_override_user_commitments.md`. <!-- LOAD-BEARING -->
- P1 DEFAULT - For destructive drops, contract changes, or retirements, produce a file:line impact map across Flutter, Worker/Edge, SQL, tests, and migrations before finalizing. Details: `feedback_three_layer_impact_map_for_audits.md`. <!-- LOAD-BEARING -->
- P1 DEFAULT - Contract-first file:line plans are mandatory for schema, RPC/API, handler, migration, auth, billing, payment, or cross-layer changes. For small localized UI/source edits, use a short plan and direct verification; do not inflate into pseudocode-level planning. <!-- LOAD-BEARING -->
- P1 DEFAULT - After audit corrections, rewrite the canonical plan inline. Do not append contradictory "Audit Amendments" sections. Details: `feedback_canonical_single_sequence_plans.md`.
- P1 DEFAULT - If the user provides a structured audit with locked decisions/directives, adopt it rather than re-interviewing. Details: `feedback_user_led_parallel_audits.md`.
- P1 DEFAULT - Keep AskUserQuestion interviews focused: 2-3 rounds, 8-12 high-leverage questions unless the user explicitly asks for more. Details: `feedback_interview_pacing.md`.
- P1 DEFAULT - Treat the canonical plan write as itself a discovery step: surface the bundled cost ledger one more time in plan form before ExitPlanMode. If the user reverses, rewrite the plan canonically (no append-amend). Details: `feedback_plan_as_mirror_for_scope_discovery.md`. <!-- LOAD-BEARING -->
- P0 HARD - Before relocating, renaming, or adding rule IDs in skill CSVs, grep the canonical CSV files at HEAD. Don't propose IDs that already exist or count files from earlier-session synthesis. Don't propose new pump/time/lifecycle/state-discipline rules without first checking `FLAKY-*` / `TASTE-*` / `CHAIN-*` for existing coverage. Rule-reference linters and coverage checkers are distinct concerns — don't conflate. Details: `feedback_verify_rule_ids_at_head.md`. <!-- LOAD-BEARING -->
- P0 HARD - Never cite a rule ID (CHIP/ANIM/etc.) without grepping the skill's `index/rules.json` at HEAD first; fabricated IDs fail `make check-rule-refs`. Details: `feedback_rule_id_grep_before_citing.md`. <!-- LOAD-BEARING -->
- P0 HARD - Before any plan declares "implement <invariant>", read HEAD source to confirm the invariant isn't already shipped. Details: `feedback_shell_already_shipped_audit_before_rewrite.md`.
- P0 HARD - Run `git log --oneline -n 25 -- <touched paths>` + `git rev-parse HEAD` at session start AND every plan revision; SessionStart hook context can be hours stale. Details: `feedback_audit_first_via_git_log_not_session_hook.md`. <!-- LOAD-BEARING -->
- P0 HARD - Before drafting a new plan file, `ls ~/.claude/plans/` and grep for subsystem keywords; if a sibling exists, continue it or surface the duplicate — do not silently fork. Details: `feedback_plan_file_prior_art_ls_before_drafting.md`. <!-- LOAD-BEARING -->
- P1 DEFAULT - Audit subagent prompts include "Step 1 of discovery: run `git log --oneline -n 25 -- <touched paths>`" so they don't audit from a stale-trajectory foundation. Details: `feedback_subagent_prompts_include_git_log_step_1.md`. <!-- LOAD-BEARING -->
- P1 DEFAULT - In plan mode, defer write-side validator runs (surface-designer pressure-test, generate, scratch artifacts) to post-approval. Read-only `validate` is OK. Details: `feedback_plan_mode_defers_write_validators.md`. <!-- LOAD-BEARING -->
- P1 DEFAULT - User instructions like "file writing waits until plan mode ends" override the default plan-file edit exception; surface plan body in chat instead. Details: `feedback_plan_mode_user_override_supersedes_plan_file_exception.md`. <!-- LOAD-BEARING -->

## Rollout And Observability

- P0 HARD - Source-level dead code is not a production defense layer. Do not apply telemetry windows to unreachable source or UI branches.
- P1 EVIDENCE-GATED - Production defense layers with real traffic must earn their cost by catching a distinct failure mode. If a reachable defense layer uniquely catches zero measured cases for 30 days, it is a deletion candidate. Details: `feedback_defence_in_depth_budget.md`. <!-- LOAD-BEARING -->
- P1 EVIDENCE-GATED - Shipping or reverting production hardening requires telemetry, probe, incident, or shipped-code evidence. This gate does not block deleting dead code or shipping small direct UI/source changes. Details: `feedback_evidence_gates_apply_recursively.md`. <!-- LOAD-BEARING -->
- P1 DEFAULT - Risky data-plane cutovers need the smallest useful evidence channel before cutover: structured logs or counters first; health endpoints, synthetic probes, and dashboards only when blast radius warrants them. Details: `feedback_observability_before_cutover.md`. <!-- LOAD-BEARING -->
- P0 HARD - Production deploy smokes whose correctness is required for the rollout MUST fail closed when prerequisites are missing — never soft-skip. Defence-in-depth probes may soft-skip; load-bearing rollout proofs cannot. Details: `feedback_production_smoke_fails_closed.md`. <!-- LOAD-BEARING -->
- P1 DEFAULT - Water-tests need bucketed evidence, usually structured logs with explicit reason fields. Durable tables and dashboards are required only when multi-day aggregation is needed. Details: `feedback_seibel_water_test_requires_bucketing.md`. <!-- LOAD-BEARING -->
- P1 DEFAULT - Prefer structured Workers Logs over speculative Analytics Engine or unused telemetry bindings. Details: `feedback_structured_logs_over_analytics_engine.md`.

## Database, Security, And Secrets

- P0 HARD - Prod is permanent, local is disposable. Local-Docker recovery plans must not bundle prod-touching steps. Every linked-mutating command needs an explicit user gate even if earlier local steps were approved. Asymmetric lifespans → asymmetric risk appetite. Details: `feedback_prod_permanent_local_disposable.md`. <!-- LOAD-BEARING -->
- P0 HARD - Never edit deployed migrations. Add a new forward migration. Details: `feedback_never_edit_migrations.md`.
- P0 HARD - Before editing or proposing edits to any existing Supabase migration, run `supabase migration list --linked` and inspect the Remote column. If the version is listed remotely, restore/leave the file immutable and create a new forward migration with `make migration-new NAME=...`. <!-- LOAD-BEARING -->
- P0 HARD - `supabase db push --linked --dry-run` validates parsing only. Dependency failures surface during apply; iterate carefully and document SQLSTATE fixes. Details: `feedback_iterate_db_push_linked_pattern.md`. <!-- LOAD-BEARING -->
- P0 HARD - Migration scope gate before any linked db push: enumerate the exact pending set (`cat LATEST_MIGRATION` + `supabase migration list --linked`) and confirm the push targets only the named migration. `supabase db push --linked` applies ALL pending migrations; if unrelated work is pending, STOP and surface to user. Details: `feedback_migration_scope_gate_before_push.md`. <!-- LOAD-BEARING -->
- P0 HARD - Before `DROP COLUMN`, enumerate and explicitly handle constraints, triggers, RPCs, views, indexes, FKs, and generated expressions. Details: `feedback_drop_column_cascade_audit.md`.
- P0 HARD - No anon-callable Supabase RPCs for user sync. Use the approved server-side path. Details: `feedback_no_anon_supabase_rpc.md`.
- P0 HARD - Service role keys are deprecated for new code. Do not introduce new service-role usage without explicit user approval and documented containment. Details: `feedback_service_role_key_deprecated.md`. <!-- LOAD-BEARING -->
- P0 HARD - Cloudflare Dashboard is the source of truth for secrets. Deploy scripts must not bulk-put/delete dashboard secrets. Details: `feedback_dashboard_is_source_of_truth_strict.md`.
- P0 HARD - Cross-surface secret parity gates (Worker↔Worker, Worker↔Firebase) verify PRESENCE only — never VALUES. `wrangler secret list` and `firebase functions:secrets:get` return metadata, not values; a gate that requires the operator to export the secret value or its SHA-256 forces a copy outside the source-of-truth surface and is worse than no gate. Operator owns same-value rotation; gate catches missing-on-one-surface only. Codified as devops-architect `SECR-CF-023`. Details: `feedback_secret_parity_presence_only.md`. <!-- LOAD-BEARING -->
- P0 HARD - For Cloudflare Worker cache-invalidation / cohort-rotation, reuse existing build-time `--var` injections (`COMMIT_SHA`, `BUILD_DATE` per `microservices/CLAUDE.md` Safe deploy template) over introducing new Cloudflare Dashboard secrets. New secrets expand audit-secrets + secrets-mapping.json + lint allowlist surface; `--var COMMIT_SHA` rotates atomically on every deploy with zero operator action. Details: `feedback_reuse_existing_var_over_new_secret.md`. <!-- LOAD-BEARING -->
- P0 HARD - Chaos/canary experiments never run on Razorpay, subscription, GST, webhook, or pg_net billing paths. Details: `feedback_chaos_never_on_billing_paths.md`.
- P0 HARD - 42501 Auth — claim merge + JWT role routing: Firebase Admin setCustomUserClaims replaces the WHOLE claims object (fetch+merge customClaims before setting role:"authenticated"); missing/no-role JWTs route to anon. Details: feedback_42501_auth_rpc_compact_doctrine.md. <!-- LOAD-BEARING -->
- P0 HARD - 42501 ACL classification + financial-RPC exception: CLOSED for financial/write/state-mutation RPCs (revoke PUBLIC/anon/authenticator, grant authenticated/service_role); SWEEP only for body-guarded user-facing; never grant authenticator. Details: feedback_42501_auth_rpc_compact_doctrine.md. <!-- LOAD-BEARING -->
- P0 HARD - 42501 withAuthRetry semantics + log redaction: read postgrestProvider INSIDE retry closure; force-refresh through SupabaseRestClient.refreshAuth (no third raw getIdToken); client [AUTH_RETRY] logs must not contain raw UID/sub/token. Details: feedback_42501_auth_rpc_compact_doctrine.md. <!-- LOAD-BEARING -->
- P1 DEFAULT - Firebase Auth V1 `onCreate` triggers do not repair claims on sign-in. Per-sign-in blocking repair requires Identity Platform; existing-user repair runs through admin/support tools.
- P1 DEFAULT - Canonical Supabase migration sentinel is `supabase/migrations/LATEST_MIGRATION` (root copy is stale). Read before proposing a number; create new files via `make migration-new NAME=...`. Details: `reference_supabase_migrations_latest_sentinel.md`. <!-- LOAD-BEARING -->
- P1 DEFAULT - Token refresh paths must call `SupabaseRestClient.instance.refreshAuth()` (singleton). `supabaseRestProvider` and `postgrestProvider` throw `StateError` on null JWT (Layer 1.5 transport guard at `supabase_provider.dart:142-159`). Details: `feedback_singleton_for_token_refresh_not_provider.md`. <!-- LOAD-BEARING -->

## UI And Product Surfaces

- P0 HARD - No UI runtime flags unless the user explicitly asks. Old banner memories that say to preserve UI paths behind flags are superseded by this rule.
- P1 DEFAULT - UI/UX changes ship as small direct diffs with visual review, widget/golden tests where useful, and fast git/hotfix rollback. Do not keep legacy UI branches for optionality.
- P1 DEFAULT - System-back / pop handlers in surfaces with debounced input must check raw controller state in addition to (or instead of) the debounced derivative; otherwise typing-then-back before debounce settles can pop the route instead of clearing input. Add a FakeAsync widget test for this race. Details: `feedback_predebounce_ui_back_handler.md`. <!-- LOAD-BEARING -->
- P1 DEFAULT - Surface-designer hard gates require a `*.surface.md` artifact at the canonical handoff location; embedded JSON inside a planning doc does not satisfy the contract. Same applies to wizard-designer, pdf-template-design, paywall-designer. Details: `feedback_surface_designer_artifact_handoff.md`. <!-- LOAD-BEARING -->
- P1 DEFAULT - Dart privacy is library-scoped. A leading-underscore widget (`_Foo`) cannot be imported from another file even with explicit `import`; declare it in the same file as its consumer, use `part`/`part of`, or make it public. Details: `feedback_dart_privacy_library_scoped.md`. <!-- LOAD-BEARING -->
- P1 DEFAULT - When a fixed bottom bar overlays a scrollable body, reserve the inset in EXACTLY one place — an `AnimatedPadding` around the body switcher — never also add bottom padding to the scrollable content (grid, list, results). Doubles space and breaks on collapse. Details: `feedback_single_source_bottom_inset.md`. <!-- LOAD-BEARING -->
- P1 DEFAULT - Dashboard ticker and summary-card leaves stay presentation-only; page/container widgets own provider reads. Details: `feedback_presentation_only_leaf_widgets.md`.
- P1 DEFAULT - If UI work changes data source, count semantics, or routing, call it information architecture and add source/routing tests. This classification does not imply a runtime flag. Details: `feedback_ia_refactor_not_pure_visual.md`. <!-- LOAD-BEARING -->
- P1 DEFAULT - Visible dashboard tickers use existing aggregation RPC/MV-backed providers; do not derive counts from rendered lists unless the user accepts drift risk. Details: `feedback_dashboard_tickers_use_mv_backed_rpcs.md`. <!-- LOAD-BEARING -->
- P1 DEFAULT - One-time paywall/upsell gates must fail-open (try/catch around launch + markShown + analytics) and mark the SharedPrefs "shown" flag ONLY when the wizard actually rendered. Coordinator preflight short-circuits (null UID, paid bypass) emit `result: 'bypassed'` with explicit `via:` discriminator and never mark shown. Details: `feedback_paywall_gate_fail_open_and_render_proof.md`. <!-- LOAD-BEARING -->
- P2 CONTEXTUAL - Banner-specific project files are historical/contextual unless actively debugging banner code. Do not copy their flag-heavy rollout posture into new UI work.

## Testing And Tooling Gotchas

- See `kernel_section_testing_tooling_gotchas.md` for 53 test-runner gotchas (Flutter Semantics types, Finder type partition, pgTAP SAVEPOINT ban, ProviderContainer discipline, withAuthRetry call-site rules, retry-policy opt-in, MediaQuery route propagation, manifest 3-level arithmetic, paywall coordinator overrides, CTA dispatch dedup, etc.). Pull rules back to kernel only if specific subsystem work is in scope this session. <!-- LOAD-BEARING -->

<!-- twinkly-parnas-sw2-applied: 2026-05-17T20:33:00Z; compaction wave applied per ~/.claude/plans/column-manager-twinkly-parnas-v5-2-fw-l-wiggly-twilight.md SW2 (L91 split into 3 entries + Testing-section body collapse to 1 pointer; verbatim Testing body → kernel_section_testing_tooling_gotchas.md). --> <!-- LOAD-BEARING -->

