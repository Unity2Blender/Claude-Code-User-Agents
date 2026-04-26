---
name: GST always-on banner contract (LD-48)
description: For placement='gst', bottomSurfaceDecisionProvider returns showBannerSyntheticFallback on null/error — NEVER showFooter*. Adding any footer path for gst breaks LD-48 and the acquisition funnel.
type: project
originSessionId: 36039f2b-32bf-4fe5-849f-bc2739e79e2d
---
**The contract:** `apps/gst_calculator/lib/state/providers/bottom_surface_decision_provider.dart:75-101` — for `placement='gst'`, the resolver returns `BottomSurfaceDecision.showBannerSyntheticFallback` on banner null/error, NEVER any `showFooter*` value.

```dart
return banner.when(
  loading: () => BottomSurfaceDecision.showPlaceholder,
  error: (_, _) => isGstPlacement
      ? BottomSurfaceDecision.showBannerSyntheticFallback
      : nonGstFallback(),
  data: (rec) {
    if (rec != null) return BottomSurfaceDecision.showBanner;
    return isGstPlacement
        ? BottomSurfaceDecision.showBannerSyntheticFallback
        : nonGstFallback();
  },
);
```

**Why:** The 3-layer always-on guarantee for the GST result page (LD-34, plan `refer-recent-implementation-plan-serene-popcorn`):
- Layer 1: DB sentinel row in `tutorial_banners` with `is_sentinel=TRUE` (retire-guard trigger raises P0001)
- Layer 2: Worker hardcoded `GST_FALLBACK_BANNER` (logged as `outcome='gst_invariant_violated'`)
- Layer 3: Client-bundled `assets/banner_fallback/gst_welcome.json` rendered by `StaticGstFallbackBanner`

The acquisition funnel depends on a tappable banner being visible on every cold start — silently falling through to a footer breaks that funnel. The `bottomSurfaceDecisionProvider` for gst is the synthesis point: it MUST route to the synthetic fallback, never to footer.

**How to apply:**
- Any test that asserts "GST shows footer on null" is **wrong**. Update assertions to `showBannerSyntheticFallback`.
- Any new code path adding `showFooter*` for gst placement is a regression — review for LD-48 violation.
- For non-gst placements (`tabBills`, `tabItems`, `tabParties`, `gstActivated` in some cohorts), the v3 fallback (sign-in / firm-setup / dual-CTA footer) is preserved.
- Tests in `apps/gst_calculator/test/views/bottom_surface/` lock this contract — keep them green.

The plan that re-discovered + re-locked this: `image-1-docs-runbooks-lib-billbook-tuto-ethereal-sparrow` (2026-04-22, D2).
