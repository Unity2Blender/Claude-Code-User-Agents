---
name: Reels View v5.2 Decisions
description: Video rendering, footer layout, CTA auth gate decisions from v5.2 overhaul (11 decisions, 4 audit agents)
type: project
---

Reels View v5.2 shipped 2026-04-09 with 11 locked decisions:

- **D-V52-01**: Flat `BillingThemeData.dark.surface` fill + `BoxFit.contain` (not cover). `surface` (#182026) is darkest M3 token.
- **D-V52-02**: CTA label auth-awareness via `ValueNotifier<bool> authStateNotifier` on `CtaResolverCallbacks` — bridges Riverpod into plugin's plain widget tree (ARCH-P02 compliant).
- **D-V52-03**: AuthTransitionSheet shown FIRST, keep re-tap (CRIT-ARCH-001). Follows established 6-callsite pattern.
- **D-V52-04**: Seekbar above title (video-adjacent).
- **D-V52-05**: Seekbar 2dp collapsed, track opacity 0.18.
- **D-V52-08**: Vignette gradient (12dp, `fillColor.withValues(alpha: 0)` — NOT `Colors.transparent` which is transparent BLACK).
- **D-V52-10**: Asymmetric spacing: 8dp seekbar→title, 12dp title→CTA.

**Why:** `Colors.transparent` (`0x00000000`) creates muddy gradient bands. Always use same-hue-zero-alpha.

**How to apply:** Any future gradient that fades to "transparent" must use the fill color with alpha 0, not `Colors.transparent`.
