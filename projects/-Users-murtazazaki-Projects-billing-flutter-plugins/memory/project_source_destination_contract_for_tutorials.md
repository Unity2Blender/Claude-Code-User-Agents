---
name: Source→Destination Contract for tutorial reels
description: Architectural principle — many sources (banners, FCM push, fallback, helper row) route to ONE destination (Reels) via source-agnostic target IDs with threaded metadata
type: project
originSessionId: 1636fa18-93d5-4e42-b373-51c0d539de25
---
**Contract:** The tutorials module has ONE destination (Reels surface — `ReelsView` + `ReelPlaceholderCard` + swipe graph) and MANY sources (GST banner, tab banners × 3, FCM push, static GST fallback asset, compact helper row, future Reels tab).

**Routing principle:** `reels.browser` is a source-agnostic entry target. It means "open reel browsing from THIS source with THESE hints" — not "open banner-specific browser." FCM push, static fallback, and degraded helper row all dispatch the same way.

**Threaded metadata (not baked into target identity):**
- `category` — which reel category to scope the playlist to (`gst`, `invoicing`, `inventory`, `parties`, etc.)
- `surface_id` — which surface's playlist (`gst`, `reels_invoicing`, `reels_inventory`, etc.)
- `source_placement` — analytics attribution (banner placement wireValue, FCM intent name, etc.)
- `source_kind` — future-proof discriminator (`banner`, `static_fallback`, `compact_helper_row`, `fcm_push`, `reels_tab`)

**Ownership split (BANNER-SRP-001 extended):**
- Source owns: provenance, hints, WHICH destination to open
- Destination owns: version gating, entitlement, prerequisite, connectivity, placeholder-open branch

**Placeholder contract (ADR-007 LD-9):** covers ALL reachable opens, not just banner RPC paths. Migration 000358 enforces on `tutorials_reels` (source of truth) that placeholder rows have non-null `category` + `cta_config` — satisfies every source→destination path (banner RPC, FCM push, static fallback, helper row).

**Anti-patterns to avoid:**
1. `banner.browser` vs `fcm.browser` vs `fallback.browser` — no source-specific target IDs. ONE target per destination, source threaded via params.
2. Baking version/entitlement checks into the banner layer — belongs on the reel's own CTA chain.
3. Policy/implementation in the source — sources only carry provenance + hints. Destination owns policy.

**Implementation references:**
- `apps/gst_calculator/lib/services/feature_navigation_coordinator.dart` — `_launchReelsBrowser` is the source-agnostic dispatcher.
- `lib/billbook/tutorials/banner/widgets/_internal/banner_compact_helper_row.dart` — example degraded-UX source.
- `apps/gst_calculator/assets/banner_fallback/gst_welcome.json` — static-fallback source with threaded params.
- `microservices/tutorials/src/routes/banner.ts:filterSchemaValidRows` — contract guard on required fields.
- `supabase/migrations/000358_tutorials_reels_placeholder_browse_open_contract.sql` — DB-side contract enforcement.

**Test references:**
- `test/billbook/tutorials/banner/widgets/banner_compact_helper_row_test.dart` — verifies threaded metadata on compact row.
- `supabase/tests/database/critical/get_banner_recommendation_all_reachable_opens_contract_test.sql` — asserts RPC output satisfies contract across all 5 placements.

**Established:** ADR-007 §2 (2026-04-22), during operator refinement of the plan. Future plans MUST NOT violate this contract — ADR-008 (pool-cohort RPC) explicitly preserves it.
