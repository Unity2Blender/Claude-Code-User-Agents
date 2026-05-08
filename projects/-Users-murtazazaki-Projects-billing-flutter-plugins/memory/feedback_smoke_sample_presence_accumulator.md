---
name: Sample-presence smokes use accumulator-then-assert, not per-row fail
description: Deploy-smoke checks that verify "at least one row carries field X" must accumulate across the loop and assert positives>0 once at the end. Per-row fail is wrong because rows legitimately lacking the sample are not regressions.
type: feedback
originSessionId: 0adda812-a90b-43e0-ba00-4a0baa85afbb
---
For deploy-smoke gates that verify "at least one returned row carries field X" (e.g., the by-id reel loop in `microservices/tutorials/scripts/verify-deploy.sh` asserting at least one reel has an allowlisted `cta_config.type`), use an accumulator pattern:

1. Initialize a counter (`BYID_CTA_OK=0`) before the outer loop.
2. Inside the loop, increment when the sample is present; do NOT increment failure flags when it is absent.
3. After the loop, assert `[[ $COUNTER -gt 0 ]]` — a single zero-positives result trips `BYID_FAIL=1`.
4. Fail-closed (`exit 1`) per kernel `feedback_production_smoke_fails_closed.md`.

**Why:** Some rows will legitimately lack the sample — a reel without a CTA, an item without a tag, a webhook without a retry header. Per-row fail flags those legitimate rows as regressions and produces false-red deploys. The actual regression mode is "the entire fleet stopped emitting field X," which the post-loop check catches with one assertion and zero false positives.

**How to apply:** Anywhere a deploy smoke loops over a representative sample to prove a contract field still exists at the response layer — DTO regressions, projection drops, codegen drift, schema misalignment. If the question is "is at least one positive?", accumulate. If the question is "are all rows valid?", per-row fail is correct.
