---
name: tutorials Worker prod — CP-26 KV "binding divergence" (RESOLVED — wrangler v4 local-default)
description: Symptom looked like CLI writing to a different KV namespace than the Worker read from. Root cause was wrangler v4 defaulting kv key put/get/delete/list to local miniflare sqlite without --remote. No binding was ever diverged. Resolution 2026-04-14 — added --remote to every wrangler kv key invocation in Makefile + scripts.
type: project
originSessionId: db9f8ed4-7e01-48cd-aec9-9508e435e39a
---
**Discovered 2026-04-13** while executing CP-26 (Phase 1 deploy gate). Must be resolved before any KV-backed flag flip (`flag:gst_v2_enabled`, `flag:banner_tab_*_enabled`, etc.) can take effect on prod.

## Symptom

1. `wrangler kv key put --env production --binding TUTORIAL_CACHE flag:gst_v2_enabled true` succeeds.
2. `wrangler kv key get --env production --binding TUTORIAL_CACHE flag:gst_v2_enabled` returns `true`.
3. `GET https://tutorials.gstapps.com/v1/tutorials/config` returns `"gst_v2_enabled": false`.
4. Worker diagnostic log (`kv.list({limit:20})`) shows keys the CLI does NOT see (e.g. `rate:undefined:2026-04-13`, `worker:diagnostic:echo`).
5. Keys the CLI writes (`smoke:probe`, `worker:diagnostic:echo` via CLI) do NOT appear in the Worker's `kv.list()`.

## What's confirmed

- **Account:** `140be945f8f3e50f77668acff1542900` per `wrangler whoami` (botasblog100@gmail.com).
- **Worker name:** `tutorial-engine-production` per wrangler.toml `[env.production]`.
- **Custom domain:** `tutorials.gstapps.com` — traffic does reach my deployed Worker (diagnostic console.log output appears in `wrangler tail`).
- **wrangler.toml binding:** `[[env.production.kv_namespaces]] binding = "TUTORIAL_CACHE" id = "3c33392b6bd34d8bbf45dd87d4bf664b"`.
- **Deploy output claims binding:** `env.TUTORIAL_CACHE (3c33392b6bd34d8bbf45dd87d4bf664b)`.
- **`wrangler versions view <latest>`** shows binding `3c33...`.
- **7 KV namespaces total in account** — none contain the Worker's writes (`rate:*`, `worker:diagnostic:echo`).

## What's NOT confirmed (needs Dashboard investigation)

- Dashboard-level KV binding override (CF UI may have edited a binding that sticks across wrangler deploys).
- Another account owning `tutorials.gstapps.com` via Worker Route pattern that takes precedence over my custom-domain binding.
- Cloudflare platform bug.

## Reproduce

```bash
cd microservices/tutorials
# 1. Deploy Worker with diagnostic dump (temporary):
#    In src/routes/config.ts, add `const listed = await kv.list({limit:20}); console.log(...)`
# 2. Deploy: npx wrangler deploy --env production
# 3. Write a key: echo "v1" | npx wrangler kv key put --env production --binding TUTORIAL_CACHE smoke:probe v1
# 4. Hit endpoint: curl -sf https://tutorials.gstapps.com/v1/tutorials/config
# 5. Tail: npx wrangler tail --env production --format=pretty
# Expected (if working): listed keys include smoke:probe
# Actual (broken): listed keys exclude smoke:probe; include worker-only writes that CLI can't see
```

## Workaround (until resolved)

- Do NOT attempt flag flips via `wrangler kv key put` — they won't take effect.
- Banner v5 Phase 1 rollout (`gst_v2_enabled`, `banner_tab_bills_enabled`) is BLOCKED until the binding divergence is resolved.
- Static Dart defaults (`defaultBannerFlags` in `banner_killswitch_provider.dart`) remain the effective source of flag state in released clients, so the current Flutter build behaves as all-flags-OFF regardless. Safe default.

## Next steps

1. **Inspect Cloudflare Dashboard** at dash.cloudflare.com → Workers & Pages → `tutorial-engine-production`:
   - Settings → Variables and Secrets → KV Namespace Bindings. Compare binding ID with `3c33392b6bd34d8bbf45dd87d4bf664b`.
   - Settings → Triggers → Custom Domains. Confirm `tutorials.gstapps.com` is claimed by this script.
   - Settings → Triggers → Routes. Check if any ROUTE pattern overlaps with the custom domain.
2. Check whether **the zone** `gstapps.com` is managed by a different Cloudflare account than `140be945f8f3e50f77668acff1542900`. Cross-account Worker Routes can intercept traffic.
3. If Dashboard shows a different binding ID: re-link to the correct namespace (UI) OR `wrangler delete --env production && wrangler deploy --env production` to force re-creation.
4. Once the binding is unified, re-run CP-26 Step 2 — `wrangler kv key put ... flag:gst_v2_enabled true` and verify via `/v1/tutorials/config` that the flag flips.

## Clean-up performed 2026-04-13

- Deleted orphan keys `flag:gst_v2_enabled` and `smoke:probe` from the CLI-visible namespace (the Worker never saw them; harmless orphans).
- Reverted `src/routes/config.ts` to clean state (no diagnostic logs, uses `isFeatureEnabled`/`isPlacementFlagEnabled` helpers).
- Deployed clean version `1a2d208a-8206-4a88-9e5e-044f37a118cc` to production.

## Blast radius

- Pre-launch, zero end-users on the released APK reference `/v1/tutorials/config`. So the broken flag-flip capability has no user impact today.
- Post-launch, this becomes the critical rollback mechanism. Fix before first Play Store public release.
- Related deploy scripts (`make phase1-gate-{test,prod}`, `phase1-deploy-gate.sh`) appear to work for test env; test env wasn't verified in this incident.

## Permanent guards landed 2026-04-13 (pending prod diagnosis + fix)

Harness shipped before the root-cause fix so the next flip-then-silently-fail is caught by tooling, not a user report:

- **Route** `microservices/tutorials/src/routes/binding-audit.ts` — `GET /v1/admin/binding-audit[?key=<k>]`, Bearer `CACHE_PURGE_TOKEN`. Round-trips a UUID KV write, lists `smoke:`+`rate:` prefixes, optionally echoes a CLI-seeded key.
- **Script** `microservices/tutorials/scripts/binding-audit.sh` — wrangler CLI writes UUID, curl reads via Worker, asserts equality, always cleans up. Exit non-zero on divergence.
- **Makefile targets** `binding-audit-test` / `binding-audit-prod` — standalone run + inserted into `deploy-test-full` / `deploy-prod-full` between `deploy-*` and `smoke-*`.
- **Pre-flight guard** in `scripts/phase1-deploy-gate.sh` — exit 3 before any flag flip if audit fails.
- **Inline step 5c** in `scripts/verify-deploy.sh` — runs audit when `CACHE_PURGE_TOKEN`+`jq`+`uuidgen` available.
- **Contract tests** `tests/contract/binding-audit.contract.test.ts` — 10 tests green (auth × 3, round-trip × 4, list prefixes × 3).
- **Docs** — `microservices/tutorials/CLAUDE.md` "KV Binding Audit Runbook (CP-26)" section with Management API `/settings` ground-truth curl + PATCH fix + "never `wrangler delete`" warning. Routes table in `src/routes/CLAUDE.md` updated.

## Resolution — 2026-04-14 — RESOLVED

**The "binding divergence" framing was WRONG.** Full diagnostic work proved:

- Management API `/workers/scripts/tutorial-engine-production/settings` → `TUTORIAL_CACHE.namespace_id = 3c33392b6bd34d8bbf45dd87d4bf664b`.
- `wrangler.toml:89` → `id = "3c33392b6bd34d8bbf45dd87d4bf664b"`.
- Domain `tutorials.gstapps.com` → correctly claimed by `tutorial-engine-production`, enabled.
- `npx wrangler kv key put --env production --binding TUTORIAL_CACHE …` printed `id: "3c33392b6bd34d8bbf45dd87d4bf664b"` in its output.
- Management API key-list on that ID → returned only `rate:undefined:*` and `worker:diagnostic:echo`. **The CLI-written key was NOT there.**
- Adding `--remote` to the SAME CLI command made the key appear on Management API instantly.

**Root cause:** `wrangler v4` changed the default for `kv key {put,get,delete,list}` from remote to **local miniflare sqlite** (`.wrangler/state/v3/kv/<id>/blobs/`). The CLI output still prints `id: "<canonical>"` making the local write look remote. Every `make set-kv-flags-prod` and `binding-audit.sh` invocation before the fix operated on the operator's laptop filesystem, never touching Cloudflare. The Worker always read from the real Cloudflare KV (correctly bound to the canonical namespace) — it just saw an empty namespace because nothing had ever been written to it.

**Fix applied 2026-04-14 (commit pending):**

1. `microservices/tutorials/scripts/binding-audit.sh` — `--remote` added to put + delete.
2. `microservices/tutorials/scripts/phase1-deploy-gate.sh` — `--remote` added to 2 puts + 2 rollback-instruction echoes.
3. `microservices/tutorials/scripts/verify-deploy.sh` — `--remote` added to kv key get.
4. `microservices/tutorials/Makefile` — `--remote` added to `set-kv-flags-{test,prod}` (both now non-destructive: pre-check with `kv key get`, only put when key MISSING, else KEEP).
5. `microservices/tutorials/Makefile` — `set-kv-flags-prod` REMOVED from `deploy-prod-full` composite (was re-seeding on every deploy, stomping operator-flipped state).
6. `microservices/tutorials/Makefile` — `purge-watch-state-prod` soft-skips on missing `CACHE_PURGE_TOKEN` (was hard-exit, broke composite).
7. `microservices/tutorials/CLAUDE.md` — root-cause note added above historical PATCH runbook; all `wrangler kv key put` examples updated with `--remote`.

**Post-fix verification (2026-04-14):**

- `make binding-audit-prod` → `OK: binding-audit env=production — CLI↔Worker round-trip succeeded`.
- Management API key-list on `3c33392b6bd34d8bbf45dd87d4bf664b` → all 5 `flag:*` keys now present (`banner_global_enabled`, `banner_tab_bills_enabled`, `banner_tab_items_enabled`, `banner_tab_parties_enabled`, `gst_v2_enabled`).
- `/v1/tutorials/config` → returns the KV-seeded values (was returning config.ts defaults before).
- Rerun of `make set-kv-flags-prod` → shows KEEP for all 5 (non-destructive confirmed).

**Preserved for future:** The 8-step Management API PATCH runbook in `microservices/tutorials/CLAUDE.md:213-336` is still correct for a real Dashboard-vs-wrangler binding split. It just didn't fire today.

**Operator:** murtazazaki + Claude Opus 4.6. **Branch fired:** wrangler-v4-local-default (new branch, not in runbook). **Stray namespace ID observed:** none — all namespace IDs were canonical throughout.
