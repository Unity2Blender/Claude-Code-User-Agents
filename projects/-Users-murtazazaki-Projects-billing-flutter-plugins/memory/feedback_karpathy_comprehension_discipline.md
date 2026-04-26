---
name: Karpathy comprehension discipline — verify HEAD, verify audits, verify deps
description: Three-layer discipline for preserving understanding 2-3 layers below where you're working — before planning, after audits, and for provider/dependency claims
type: feedback
originSessionId: 1636fa18-93d5-4e42-b373-51c0d539de25
---
**Rule:** Before incorporating audit findings or proposing changes that rest on claims about the codebase, verify the claims at the source via `Read`/`Grep`. Karpathy guardrail: "Abdicate the toil, not the thinking. Understand systems 2-3 layers below where you're working."

Three compounding sub-rules — apply in the phase that matches your work:

## 1. Verify HEAD before planning

When spawning multi-agent audits to scrutinize a plan, do not blindly trust their findings. Pick the 3-5 highest-stakes claims that materially change the plan's shape and verify each via `Read`/`Grep` against live HEAD. Stale claims warrant rewrite, not patch.

**Why:** Plan `image-1-docs-runbooks-lib-billbook-tuto-ethereal-sparrow` (2026-04-22) had 9 directives resting on stale assumptions — hero height (claimed 75dp; live is 80dp via LD-12), surface tone source (claimed widget reads `bannerBgColor`; v5 already retired via `BannerSurfaceResolver`), trailing chevron (claimed needed adding; already shipped at `banner_content_icon.dart:66-72`), playlist RPC migration (claimed 000303; live is 000238), `/v1/admin/banner-pool-debug` endpoint (claimed existed; live is `/v1/tutorials/banner/diagnose/:placement`), `/v1/tutorials/config` TTL (claimed 300s; verified 60s at `config.ts:105`), GST fallback contract (claimed footer fallback; live is always-on synthetic per LD-48), goldens count (claimed 30 generic; verified 30 = Mode A 12 + Mode B 12 + Mode C 6 shrinkable), Phase 4 indexes (proposed CONCURRENTLY; conflicts with all-in-transaction migration posture). 5 minutes of `Read`/`Grep` would have caught all 9.

**How to apply:**
1. After audits return, identify the top 3-5 findings that change the plan's structure.
2. Direct `Read`/`Grep` against HEAD for each.
3. Document verification with file:line citations in the plan's Context section.
4. Applies strongest to: migration file numbers, widget code paths, endpoint existence, test counts, already-shipped behavior.

## 2. Verify audit claims at the source

When audit agents return findings, verify the highest-stakes claims by reading the actual source (file:line, migration body, schema DDL) before accepting them wholesale. Audit reports distill what the tool found but can still be wrong or misleading. Time spent verifying is NOT wasted.

**Why:** On 2026-04-22 during ADR-007 plan authoring, operator sent the Karpathy comprehension-debt guardrail. Plan v1 had a WRONG-TABLE CHECK constraint (targeted `tutorial_banners.title` / `cta_config` / `publication_status` — columns that live on `tutorials_reels`, not `tutorial_banners`). The design-postgres-tables audit caught this, but operator pointed out I should have caught it myself by reading `000357:87` (SELECT FROM `tutorials_reels t` for title/category/cta_config — 100% proof the columns don't live on the banner table).

**How to apply:**
1. After any audit returns findings, pick the 3-5 highest-stakes claims and verify at source.
2. For schema claims: read the `CREATE TABLE` DDL or the SELECT list of RPCs that read the table.
3. For code claims: read the function body + its dependencies. Chain-of-reads beats trusting the agent's chain-of-thought.
4. For "X doesn't exist" / "Y is already handled" claims — verify BOTH sides.
5. Example from ADR-007 v2: audit said "Worker COALESCE is dead code — RPC 000357:87 already does it." Verified by reading `000357:87` line-by-line → confirmed. Removed the Worker COALESCE from the plan. The v1 plan had a FATAL blocker (SQLSTATE 42703 on migration apply). Catching via audit cost <2 min; verifying cost <1 min; accepting without verification would have wasted a deploy cycle + rollback.

## 3. Verify actual dependency edges (not coincidence)

When a bug manifests as "provider A reads stale value from provider B," verify A actually depends on B by reading A's body (the `@riverpod` function and its `ref.watch`/`ref.read` calls). Correlation in logs ≠ causation in the dependency graph — two providers can both depend on a common upstream (e.g., `jwtStateProvider`) and race independently without one depending on the other.

**Why:** On 2026-04-22 during ADR-007 plan authoring, v2 proposed adding `await ref.read(billingContextProvider.future)` to `banner_prewarm_notifier.dart` BEFORE reading `userLightweightSummaryProvider.future`. Hypothesis: "prewarm reads summary which reads billing context; await billing context first so summary sees resolved firm." Operator caught this: `userLightweightSummaryProvider` at `lib/state/providers/user_lightweight_summary_provider.dart:176-257` reads `backendReadyProvider`, `jwtStateProvider`, `userContextProvider`, and `postgrestProvider` — it NEVER reads `billingContextProvider`. The correlation was coincidence (both auth-dependent), not causation. The extra await was a no-op for the actual summary call chain.

**How to apply:**
1. Before proposing "add `await ref.read(X.future)` to fix timing," READ X's provider function and identify what it actually watches. Grep the file for `ref.watch` and `ref.read` calls.
2. If the dependency is indirect (X watches Y watches Z), the fix should be at the RIGHT layer — adding an await at a layer that doesn't depend on the awaited provider is a no-op that hides the real gap.
3. For prewarm-style timing bugs, instrument the actual edges first (wall-clock timing JWT → backendReady → summary → fanout) so the real delay becomes visible. Then fix the specific slow edge. ADR-007 LD-7 chose this path: added `summaryTimeout` + `summaryResolved` telemetry hooks to `banner_prewarm_telemetry` instead of the wrong-edge await.
4. Comprehension-debt rule applies 2-3 layers DOWN and also 2-3 layers OVER in terms of dependency graphs. Read the provider chain, don't guess it.

**Sources:**
- §1 from plan `image-1-docs-runbooks-lib-billbook-tuto-ethereal-sparrow` (2026-04-22) — originSessionId 36039f2b-32bf-4fe5-849f-bc2739e79e2d.
- §2 from ADR-007 plan revision log v1 → v2 → v2.1 (direct-reads verification).
- §3 from ADR-007 plan revision log v2.2 → v3.0 — operator note: "banner_prewarm_notifier.dart:249-273 assumes userLightweightSummaryProvider depends on billingContextProvider, but user_lightweight_summary_provider.dart:176-257 does not. Instrument/test JWT-summary timing instead."
- Karpathy "Beware of Comprehension Debt" guardrail sent by operator 2026-04-22.

**Supersedes:**
- `feedback_verify_head_before_planning.md` (consolidated as §1)
- `feedback_comprehension_debt_verify_audit_claims_directly.md` (consolidated as §2)
- `feedback_verify_actual_dependency_edges_not_coincidence.md` (consolidated as §3)
