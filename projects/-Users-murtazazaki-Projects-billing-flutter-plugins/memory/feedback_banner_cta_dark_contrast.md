---
name: GOTCHA-029 banner CTA dark mode contrast trap
description: Never use cs.error as CTA foreground on cs.errorContainer — 1.45:1 ratio in dark mode. Always use onErrorBannerContainer.
type: feedback
originSessionId: b352baea-be19-4b54-a872-17f086754485
---
GOTCHA-029: Never use `colorScheme.error` (#C4454D) as CTA foreground on `colorScheme.errorContainer` (#7F1D1D dark). Same-hue creates near-zero contrast — only 1.45:1 ratio (WCAG AA requires 4.5:1).

**Why:** Discovered in the CRITICAL (loss-leader) banner — "Yes, intentional" and "Fix cost" buttons were invisible in dark mode. The WARNING and NOTICE banners didn't have this issue because they use `BillingThemeData` tokens (`onWarningContainer`, `onInfoContainer`), but CRITICAL used stock M3 `cs.error`.

**How to apply:** For all banner CTA text, use the matching `onXxxBannerContainer` token from `BillingThemeData`:
- Error banners: `billingTheme.onErrorBannerContainer` (#E08080 dark, 5.7:1 on #3D2020)
- Warning banners: `billingTheme.onWarningContainer` (already correct)
- Info banners: `billingTheme.onInfoContainer` (already correct)

Rule codified as BANNER-002 and GOTCHA-029/030 in the ui-styling skill.
