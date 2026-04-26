---
name: Banner heights are placement-bound (60dp tab / 72dp GST)
description: Tab banner placements render at 60dp; GST result page placements at 72dp; surface containers enforce. Overrides ADR-010 v7.1 amendment that wrongly locked at 72dp for both.
type: feedback
originSessionId: 88479857-ca35-4a9f-bd6b-ef7c16840ff0
---
**Rule:** All tab placements (`BannerPlacement.tabBills | tabItems | tabParties`) render at **60dp** total height. All GST result page placements (`BannerPlacement.gst | gstActivated`) render at **72dp**. The widget itself is height-agnostic; the surface container that wraps it (tab wrapper or GST result page bottom surface) enforces the height via `SizedBox(height: <placement-height>)`. Applies to all banner-slot widgets: `TutorialBannerCard`, `TutorialTabBannerCard`, `BannerCompactHelperRow`, `BannerSlotPlaceholder`, `StaticGstFallbackBanner`.

**Why:** Operator locked this 2026-04-24 with explicit directive: "60 DP banners' heights are supposed to be for tab banners. 72 DP height banners are supposed to be for GS result page... differentiating in widget test as well as the implementation in the surface bound which over containers which wrap them. Make this the final obvious right things, no ambiguity."

This OVERRIDES ADR-010 v7.1 amendment (commit `4c9c0e9f`) which incorrectly locked `tabBannerHeight = 72.0` and claimed the v7 plan's "60dp" was inverted. The constant change in v5.1 RD-1 (60→72 to "host subtitles without clipping") is reverted; tabs do NOT need subtitles per content design. v5 LD-12 (gst hero locked at 80dp) is also reverted to 72dp for visual consistency with the placement-bound rule.

Cascade required when implementing:
- `BannerClampConstants.tabBannerHeight: 72.0 → 60.0`
- `BannerClampConstants.gstBannerHeight: 80.0 → 72.0`
- Illustration clamps re-derive: `tabIllustrationClamp = Size.square(44)`, `gstIllustrationClamp = Size.square(56)`
- 7 sentinel test assertions in `banner_clamp_constants_test.dart`
- 4 height-pinning tests in `tutorial_banner_card_test.dart` + `tutorial_banner_card_height_test.dart`
- `lib/billbook/tutorials/banner/CLAUDE.md` v3 history table + v5 entries

**How to apply:** Before any future audit cycle re-claims `tabBannerHeight=72` or `gstBannerHeight=80` as "correct", consult ADR-010 §v8 HEIGHT-PLACEMENT-LOCK-001 + this memory. The constants in `banner_clamp_constants.dart` may NOT yet reflect this lock if PR-β-bundle has not yet shipped — read `git log -- lib/billbook/tutorials/banner/utils/banner_clamp_constants.dart` to verify whether the cascade refactor landed before relying on the live constant values.
