---
name: Pre-launch testing — release build against prod endpoint is intended
description: Before the first Play Store rollout, user tests Workers/services directly on prod. Don't route them through test-env.
type: feedback
originSessionId: db9f8ed4-7e01-48cd-aec9-9508e435e39a
---
When a Worker or cloud service is behind a feature gate that has **no real users yet** (e.g. Banner v5 pre-v11.0-launch), the user's preferred workflow is:

1. `npx wrangler deploy --env production` (or `make deploy-prod`) straight to the prod endpoint.
2. Build the Flutter **release** (or profile) build pointed at the `*_PROD_URL` env var.
3. Verify on their own device — they are the sole traffic, so prod behaves like a private staging.
4. Flip KV flags on prod directly (`wrangler kv key put --env production ...`) as needed.

**Why:** release build is the "final environment" — debug build has different tree-shaking, different ProGuard paths, different asset bundling. Testing only in debug would miss release-only regressions. With zero end-users on prod behind a false-default flag, there's no downside to using prod as the first real test.

**How to apply:**
- For deploy suggestions on pre-launch features: default to prod Worker, not test-env.
- Don't insist on a "test env dry-run first" ceremony when the user is the only possible traffic. A single gate flip + verify on a release build is enough.
- Once a feature is behind a truly-disabled-for-users flag (default-FALSE KV + no client build shipped that references it), prod is as safe as a staging env.
- Keep the Makefile composite flows (`make deploy-prod-full` = flags + purge + deploy + smoke) but also accept `npx wrangler deploy --env production` directly when the user just wants a code-only redeploy.

**Flips to the opposite stance the moment v1.0 is live on Play Store.** See `project_tutorials_worker_testenv_carveout.md` for the carve-out TODO.
