---
name: Post-launch carve-out — separate tutorials-test from tutorials.gstapps.com usage
description: Once v1.0 ships to Play Store with real users, stop using prod Worker for dev/QA; start using tutorials-test.gstapps.com as real pre-prod.
type: project
originSessionId: db9f8ed4-7e01-48cd-aec9-9508e435e39a
---
**Status (as of 2026-04-13):** the tutorials Worker has two environments — `tutorials-test.gstapps.com` and `tutorials.gstapps.com` — but the user tests banner rollouts directly on prod because it's pre-launch (no real users yet). This is deliberate — see `feedback_prelaunch_direct_prod_testing.md`.

**Trigger to flip this model:** first public release of the GST Calculator / billing app that reaches the Play Store Production track and has ≥ 1 non-team user. At that point prod behavior is user-facing, and dev-verification on prod is no longer safe.

**Why:** Once real users hit `tutorials.gstapps.com`, a pre-release verification still needs to happen somewhere. The existing infra already supports it:
- Flutter debug builds read `TUTORIAL_ENGINE_LOCAL_URL` (via `TutorialEngineConfig`) — point this at `tutorials-test.gstapps.com`.
- `make deploy-test` + `make smoke-test-worker` already deploy + verify the test env.
- `/v1/tutorials/config` (CP-23) will return flags from whichever env's KV is configured — so the debug build reads test-env flags, the release build reads prod-env flags.

**How to apply (when the trigger fires):**
1. Update `microservices/tutorials/CLAUDE.md` GST Channel 3 rollback section: insert a "Step 0 — test-env smoke" before "Step 1 — prod flip". Flip flags on `tutorials-test` first, point a debug APK at the test env, verify banner renders + Firebase DebugView fires events, then promote to prod.
2. Add the `/v1/tutorials/config` smoke check to a pre-deploy CI gate (or a local `make pre-release-check`) so engineers can't deploy a client build whose config-endpoint shape diverges from the deployed Worker.
3. Audit all `*-PROD_URL` env vars in `flutter.env` — confirm they're used only in release/profile builds; debug builds should hit `*-LOCAL_URL` (tunnel or test Worker).
4. Consider a `test-banner-rollout` device flavor in VSCode launch.json that forces `TUTORIAL_ENGINE_LOCAL_URL` → `tutorials-test.gstapps.com` so QA can verify a flag flip on test without changing env files.

**Where this TODO is parked:** `microservices/tutorials/CLAUDE.md` → "Banner v5 Content-Ops Runbook" → new section "Post-launch environment carve-out".

**Adjacent concern — other services:** the same stance applies to every microservice gated behind a pre-launch feature. When the app ships, audit `bill-ocr`, `payment-gateway`, `whatsapp` (if live), and `supabase-proxy` for the same pre-prod discipline. Most already use test env properly (bill-ocr has `bill-ocr-test.gstapps.com`, payment-gateway has `payment-dev.gstapps.com`) — just confirm.
