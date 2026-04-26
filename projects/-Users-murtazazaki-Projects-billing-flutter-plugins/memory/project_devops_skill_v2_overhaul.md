---
name: DevOps Architect Skill v2.0 Overhaul
description: Cloudflare-first overhaul of devops-architect skill — archived GCP rules, added 6 new Cloudflare domains, updated agent
type: project
---

DevOps Architect skill overhauled from GCP Cloud Run-centric to Cloudflare Workers-first (v2.0, 2026-04-07).

**Why:** Production stack fully migrated to Cloudflare Workers (6+ Workers deployed). ~169 GCP rules were for unused technologies (Cloud Run, Cloud Build, K8S, GCP Secret Manager).

**How to apply:**
- 556 active rules across 21 CSVs (was 717 across 20)
- 171 archived rules in `data/rules/archive/` (not deleted — retrievable)
- 6 new domains: CFQ (Queues), MWK (Multi-Worker), HONO (Hono Framework), CFTUN (Tunnel), WOBS (Workers Observability), FBAW (Firebase Auth Workers)
- service-designer-lead agent updated for Cloudflare-first biases
- SKILL.md + CLAUDE.md updated to v2.0

**Remaining phases (future sessions):**
- Phase 4: Encode 36 audit findings as rules (distributed into existing CSVs)
- Phase 5: Update existing rules (PAY, RES, DEBUG, GHACT, CFW)
- Phase 6: Rewrite decision trees (service_type_selector Workers-default)
- Phase 7: Add 6 Cloudflare templates (Hono scaffold, queue consumer, etc.)
- Phase 8: Add/rewrite 6 resource docs

**Audit findings (3 critical fixed):**
- supabase-proxy: CORS fallback + Content-Length spoofing
- payment-gateway: HMAC self-comparison clarity
- 33 remaining findings to encode as rules

**Plan file:** `.claude/plans/cached-roaming-balloon.md`
