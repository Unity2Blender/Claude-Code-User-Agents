---
name: Workers Logs over Analytics Engine for developer telemetry
description: For CF Worker microservices with developer-only diagnostic needs, default to structured Workers Logs (console.log(JSON.stringify({event:'...'}))) — never add Analytics Engine bindings speculatively.
type: feedback
originSessionId: 848b4e9a-5fa8-49c6-bf7c-034f89063180
---
Default to structured Cloudflare Workers Logs over Analytics Engine for every new Worker microservice. Every log MUST be `console.log(JSON.stringify({ event: '<snake_case_event>', request_id, ...fields }))` via `src/utils/logger.ts`. Analytics Engine requires explicit justification per devops-architect rule WOBS-010.

**Why:** On 2026-04-22 the tutorials-engine prod deploy (`make deploy-prod`) failed with CF error 10089 — `env.ANALYTICS` bound to a dataset (`tutorial_engine_banner_absence_production`) that was never enabled on the Cloudflare account. An audit found the binding was used at exactly one call site, defensively null-checked, fire-and-forget, with **zero active consumers** — no Grafana dashboard, no pagerduty hook, no tests mocking it. The runbook SQL template was manual-only. The 3-month AE retention was speculative — the team had no historical-audit requirement at pre-launch. Meanwhile Cloudflare shipped the Workers Observability UI refresh on 2026-02-06 that expands full structured JSON inline + supports field-level queries, closing the "only error code + title" gap that historically motivated reaching for AE. The deploy unblocked in <1h by removing the binding and swapping `env.ANALYTICS.writeDataPoint(...)` for `logger.info('banner absence reported', { event: 'banner_absence_reported', ... })`.

**How to apply:**
- **New Worker microservices** default to `[observability] enabled = true` + `src/utils/logger.ts` with structured JSON + top-level `event` field. Do NOT add `[[analytics_engine_datasets]]` to wrangler.toml unless all three WOBS-010 criteria are documented: (a) >3mo retention hard requirement AND Logpush→R2 can't satisfy it, (b) user-facing analytics dashboards needing SQL aggregation, or (c) volume exceeding 5B/day cap.
- **When swapping an existing AE binding to Workers Logs**, follow the migration checklist in `.claude/skills/devops-architect/resources/workers_observability_patterns.md` §8 — grep all `env.<BINDING>` sites, verify defensive null-check, delete wrangler block + Env field + imports, add contract test locking the `event` literal + payload fields, update runbooks.
- **Quarterly AE audit** — any existing AE binding with zero query-read activity in 90 days must be retired.
- **Before reaching for AE** for a real need, first evaluate Workers Trace Events Logpush → R2 (~$1/mo, unlimited retention via lifecycle rules, grep-able compressed JSON). Tracked in each Worker's `docs/observability_roadmap.md`.
