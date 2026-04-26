---
name: CTA Feature Target Registry
description: 12 locked architectural decisions for tutorial reels CTA navigation — semantic FeatureTargetRef contract, module manifests, stale-target WhatsApp fallback
type: project
originSessionId: 29ba6916-0dc5-4620-8e42-d5ad710f2971
---
Tutorial reels CTA navigation system uses semantic `FeatureTargetRef` (dot-separated IDs like `invoice.new.sales`, `payments.record.in`) instead of raw URI paths. 12 locked decisions from 6-agent audit (2026-04-09).

**Why:** Raw URI targets drift from code; the stable boundary is the entry-point layer (`lib/navigation/entry_points/`), not literal paths.

**How to apply:**
- New CTA authoring uses `target_id` + `target_params` in JSONB, NOT raw URI paths
- Legacy URI strings handled via `LegacyTargetAdapter` (migration bridge only)
- Each feature module owns its target descriptors; app composes into one registry
- `FeatureNavigationCoordinator` (app layer) adapts targets to existing entry points — NOT a new router
- Post-auth: re-tap required (CRIT-ARCH-001)
- Post-paywall: auto-navigate (optimistic dispatch)
- Unknown target fallback: min_build guard → update check → WhatsApp developer contact (NEVER silent no-op)
- Reports v1: preset targets + picker flows (no entity deep links)
- `package_info_plus` added to GST Calculator for real build number
- Plan file: `.claude/plans/delightful-sniffing-rivest.md`
