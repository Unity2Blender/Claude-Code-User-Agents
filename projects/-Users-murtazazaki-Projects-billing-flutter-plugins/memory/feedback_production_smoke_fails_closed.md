---
name: Production smoke fails closed
description: Production deploy verification smokes whose correctness is required for the rollout MUST fail closed when prerequisites (e.g., TEST_FIREBASE_JWT) are missing — never soft-skip.
type: feedback
originSessionId: 2c5fcb68-16a2-4df2-b8c9-85d16216119a
---
P0 HARD — When a deploy verification smoke is the load-bearing proof for a production rollout, missing prerequisites must fail the script, not soft-skip with a warning.

**Why:** 2026-05-02 plan rejection. `microservices/tutorials/scripts/verify-deploy.sh:136` currently echoes `"SKIPPED — TEST_FIREBASE_JWT/FIREBASE_TEST_JWT not exported"` and returns success. For everyday deploys this is acceptable (operator may not have rotated a 1h-old JWT). But for the videos-tab search forward fix specifically, the authenticated `/v1/tutorials/search` smoke IS the production proof — without it, we don't know if the deploy actually fixed the regression. A soft-skip means an operator can deploy a broken Worker and verify-deploy.sh prints "ALL GREEN" because the relevant probe was skipped.

**How to apply:**
- Distinguish two classes of deploy smoke:
  1. **Defence-in-depth probes** — soft-skip on missing prereq is acceptable (e.g., KV round-trip when CACHE_PURGE_TOKEN absent in test env).
  2. **Load-bearing rollout proofs** — fail closed when prereq absent (e.g., authenticated search smoke when the rollout is a search RPC contract change).
- The plan owning the rollout decides which class each smoke belongs to. Default for novel rollouts: fail closed.
- If a smoke is reclassified to load-bearing for one rollout, surface it in the rollout doc and revert after the soak window if generic deploys would suffer.
- For the search rollout: q='import parties' (cross-surface launchable), q='parties.import' (CTA target_id alias), q='33' (id_exact branch) all fail closed on missing JWT, with explicit operator instructions to run `bash microservices/tutorials/scripts/refresh-test-jwt.sh` before deploy.
