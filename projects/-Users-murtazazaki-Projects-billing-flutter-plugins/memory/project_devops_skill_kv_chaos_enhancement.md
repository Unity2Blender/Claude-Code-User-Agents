---
name: DevOps skill KV + chaos enhancement (2026-04-18)
description: devops-architect skill v2.1 adds 2 new CSVs (KV-RACE + STATE-CHOICE, CHAOS + DOGFOOD + CANARY), rotates CFW-064, extends storage_selector with write-pattern branch, adds 4 templates and a reliability_frontier.md reference doc
type: project
originSessionId: 36859de4-f174-4b1e-a38b-23ec596500f8
---
## What shipped

Plan file: `/Users/murtazazaki/.claude/plans/claude-skills-devops-architect-claude-s-declarative-locket.md`

- **New CSVs** in `.claude/skills/devops-architect/data/rules/`:
  - `kv_concurrency.csv` — 12 rules (KV-RACE-001..006, STATE-CHOICE-001..006)
  - `chaos_progressive.csv` — 11 rules (CHAOS-GUARD-001, CHAOS-001..005, DOGFOOD-001..002, CANARY-001..003)
- **Rotated** the unsafe CFW-064 rule (was recommending KV rate-limit counters -- DEPRECATED, now points to STATE-CHOICE-001)
- **Extended** `storage_selector.dart` with `WritePattern` + `ConsistencyNeed` enums and `decideConcurrentState()` method plus 5 new `StorageTarget` values (cloudflareKV, durableObjectsSqlite, cloudflareD1, cloudflareQueues, cloudflareHyperdrive)
- **4 new templates**: `worker_do_counter.ts`, `worker_kv_safe_reader.ts`, `worker_queue_producer.ts`, `chaos/k6_load_profile.js` (with CHAOS-GUARD-001 runtime refuse-list)
- **Resource doc**: `resources/reliability_frontier.md` (storage matrix, chaos lineage, guard-rail scope, k6 quickstart)
- **Skill bumped** 2.0.0 -> 2.1.0 (CLAUDE.md + SKILL.md updated)
- **BM25 index rebuilt** — 619 total active rules

**Why:** Two gaps in the skill were biting this project -- (a) KV was being recommended for concurrent-writer patterns (rate-limit counters; CFW-064 was the smoking gun) despite Cloudflare's explicit last-write-wins semantics, and (b) the resilience rules were 100% prediction-first with zero "turn on the water" chaos-dogfood counterweight. The 2025-2026 storage landscape (Durable Objects SQLite GA 2025-04-07, Hyperdrive free-tier 2025-04-08) wasn't captured either.

**How to apply:**

- **When choosing storage**: run `dart .claude/skills/devops-architect/scripts/decision-trees/storage_selector.dart --write-pattern=<single|concurrent|fan-in|txn> --consistency=<eventual|strong|acid|none>`. Concurrent writers + strong consistency routes to Durable Objects SQLite (STATE-CHOICE-001), not KV.
- **When planning a rollout**: consult `chaos_progressive.csv` (CANARY-001/002/003). First verify the path is NOT in CHAOS-GUARD-001's protected scope (`resources/reliability_frontier.md` has the full list). If the path is billing/payment/webhook, use a deterministic test-env dry-run instead -- see `feedback_chaos_never_on_billing_paths.md`.
- **When using KV**: check STATE-CHOICE-005 first. KV is safe only for read-heavy, cacheable, staleness-tolerant, single-writer-per-key paths. If any of those four don't hold, pick a different primitive.
- **When migrating existing KV counters** (e.g., any code still referencing CFW-064): follow STATE-CHOICE-006's 5-step dual-write cutover -- don't switch in a single deploy, you'll lose increments during the race.
