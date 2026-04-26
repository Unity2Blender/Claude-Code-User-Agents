---
name: Dashboard is primary source of truth for Worker secrets — STRICT enforcement
description: Deploy-time Makefile targets MUST NEVER write secrets. Only explicit secrets-setup-* wizards may. Rule extends prior dashboard-first guidance with 7 enforceable invariants.
type: feedback
originSessionId: 00d5ac66-32db-499b-b97c-b4ba118a188a
---
**Rule** — the Cloudflare Dashboard is the PRIMARY SOURCE OF TRUTH for every Worker secret. `deploy-*` Makefile targets (and `npx wrangler deploy`, CI scripts, any automation) MUST NEVER invoke `wrangler secret put/bulk/delete`, directly or transitively. Only explicit wizard targets like `make secrets-setup-{worker}` may write secrets, and they must be interactive (`read -rs` TTY prompt), never `.env`/`.dev.vars`-driven.

**Why:** The user was explicitly, angrily adamant about this on 2026-04-14 when recapping a Makefile audit for the webhook-worker. Their fear: a freshly-cloned repo without local `.dev.vars`, or a dev's stale secrets being bulk-uploaded during a deploy, would silently overwrite live production secrets. The 2026-03-14 payment-gateway v10 incident (trailing-newline HMAC rejection) demonstrated the blast radius: 100% webhook rejection, 24h → auto-disable, silent revenue loss. OpenAI Codex would review the enforcement — this was treated as an operational principle, not a preference.

**7 invariants** (enforced via `devops-architect` skill rules SECR-CF-013 through SECR-CF-019):
1. Deploy targets MUST NEVER call `wrangler secret put/bulk/delete`
2. Only `secrets-setup-*` wizards write secrets (interactive TTY only)
3. Every wrangler config MUST have `keep_vars: true` (SECR-CF-009 already enforces)
4. `wrangler secret put` MUST use `printf '%s'`, NEVER `echo` (trailing-newline corruption)
5. `audit-secrets.sh` is read-only — never auto-fixes
6. No `.github/workflows/` secret-setting (manual Makefile deploys only)
7. Secrets-to-vars migration: deploy-first, delete-last

**How to apply:**
- When scaffolding a new microservice Makefile, copy the safe template from `microservices/CLAUDE.md`'s "Secrets Handling Policy" section
- Before approving a PR that touches a Makefile, run: `grep -rE "wrangler\s+secret\s+(put|delete|bulk)" microservices/**/Makefile | grep -v 'secrets-setup\|secrets-prod\|secrets-test'` — must return 0 hits
- Every `wrangler.toml/jsonc` MUST have `keep_vars: true` — check before deploy
- Orphan scripts like `bill-ocr/scripts/secrets.sh` and `tutorials/scripts/secrets.sh` (unreferenced, use `echo` pipes) are candidates for deletion, not resurrection
- Explicit wizard targets use `wrangler secret bulk` for atomic multi-secret updates (SECR-CF-012) — this is the CORRECT place to do bulk ops

**Current state (2026-04-14 audit):** Zero UNSAFE findings across payment-gateway (4 Workers), bill-ocr, tutorials, supabase-proxy. Codified in `microservices/CLAUDE.md` + `devops-architect/data/rules/secrets.csv`.
