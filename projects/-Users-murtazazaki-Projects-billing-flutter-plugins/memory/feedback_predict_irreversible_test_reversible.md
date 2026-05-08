---
name: Predict irreversible, water-test reversible
description: Split YC turn-on-the-water doctrine by reversibility — predict + guard irreversible pipes before deploy, only run real-traffic exposure on reversible runtime behavior with bucketed logs and rollback thresholds.
type: feedback
originSessionId: 2c5fcb68-16a2-4df2-b8c9-85d16216119a
---
P0 HARD — Apply YC Michael Seibel's "turn the water on high" philosophy ONLY to reversible runtime behavior. For irreversible or hard-to-revert pipes, predict and guard before deploy.

**Why:** 2026-05-02 plan rejection. I framed the videos-tab search forward fix as a uniform "turn on the water" experiment. The user split it correctly: schema migrations, ACL grants, RPC contract changes, KV cohort key shapes, and migration ledger writes are all hard or impossible to reverse cleanly — they need pre-deploy gates (pgTAP, dry-run, ledger inspection, scope gates). Only post-deploy runtime behavior (Workers Logs distributions, error rates, latency, cache hit/miss patterns) is the appropriate place for water-test bucketed evidence.

**How to apply:**
- IRREVERSIBLE / hard-to-revert pipes — predict + guard:
  - Forward migrations (schema, RLS, function bodies, generated columns) → pre-flight `supabase migration list --linked`, dry-run, pgTAP probes for proconfig + signature + ACL + index survival, explicit USER GATE before push.
  - ALTER COLUMN ... SET EXPRESSION on STORED columns → document the table rewrite + GIN rebuild + lock window + statement_timeout posture in a banner block.
  - Worker contract changes (route shape, response envelope, auth gate) → pre-deploy contract tests + smoke on a non-prod build + USER GATE.
  - KV cohort key shape changes → only via cache key version bump + Worker redeploy (instant invalidation), not in-place mutation.
  - ACL grants (REVOKE/GRANT EXECUTE) → critical-tier pgTAP enumerating exact role membership.
- REVERSIBLE runtime behavior — water-test with bucketed evidence:
  - Workers Logs distributions: `match_kind`, `corpus_filters_applied`, `response_size_bytes`, `kv_get_ms p99`, `rpc_ms p99`, `5xx rate`, `row_count=0 rate`.
  - Flutter `debugPrint` ring buffer for query length, build-number propagation, debounce settle distribution.
  - Rollback thresholds named explicitly with baseline + trigger window.
- The split is the load-bearing doctrine, not the philosophy itself. Plans that uniformly apply "turn on the water" without classifying by reversibility will be rejected.
