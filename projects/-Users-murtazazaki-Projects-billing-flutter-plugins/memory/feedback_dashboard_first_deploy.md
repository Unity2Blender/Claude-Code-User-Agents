---
name: Dashboard-First Deploy Philosophy
description: User deploys manually via Makefile, rejects GitHub Actions/CI, Cloudflare Dashboard is source of truth for secrets
type: feedback
---

Cloudflare Dashboard is the source of truth for all secret values. wrangler.toml/jsonc `[vars]` sections are config-only (ENVIRONMENT, feature flags, cache TTLs).

**Why:** Pre-keep_vars, empty placeholder vars in `[vars]` sections overwrote real Dashboard values on deploy. After `keep_vars=true` fix, user wants additional guardrails.

**How to apply:**
- NEVER suggest GitHub Actions, environment protection, or CI/CD pipelines for deploys
- NEVER put SECRET/TOKEN/PASSWORD keys in wrangler.toml `[vars]` (SECR-CF-011)
- All deploys are manual: `make deploy-*` after local tests pass
- Interactive secrets wizard: `make secrets-setup-<worker>` for guided setup
- `make lint-wrangler` runs as pre-flight gate on every deploy
- GitHub workflow file was deleted entirely (commit pending)
