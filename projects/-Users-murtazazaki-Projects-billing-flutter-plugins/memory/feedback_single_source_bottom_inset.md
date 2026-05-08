---
name: Single-source bottom inset — AnimatedPadding around body, not double-padding on lists
description: When adding a fixed bottom bar (action bar, banner, sticky CTA) over a scrollable body, reserve the height ONCE via an AnimatedPadding around the body switcher — never also add bottom padding to the scrollable content (grid, list).
type: feedback
originSessionId: 5fbffcbe-4672-4cdb-a9ca-c76034c28dfa
---
When a Flutter screen has a body that may contain a scrollable + a persistent bottom overlay (action bar, sticky banner), reserve the bottom inset in **one** place:

```dart
Stack(
  children: [
    Positioned.fill(
      child: AnimatedPadding(
        duration: barDuration,
        padding: EdgeInsets.only(bottom: barCollapsed ? 0 : kBarReservedExtent),
        child: body,            // PageTransitionSwitcher / SingleChildScrollView / ListView
      ),
    ),
    Positioned(left: 0, right: 0, bottom: 0, child: BottomBar(...)),
  ],
)
```

**Anti-pattern:** Adding `padding: EdgeInsets.only(bottom: 80)` to the inner `SingleChildScrollView` / `ListView` AS WELL AS the outer Stack's padding. Doubles the reserved space, scrolls poorly under reduced motion, and silently breaks when the bar collapses (inner padding still reserves space the bar no longer occupies).

**Why:** Round-4 audit on `videos-tab-majestic-harbor` flagged a plan that reserved bottom space in the Positioned.fill Padding AND the category grid AND the search-results list. Single-source via `AnimatedPadding` solves all three at once and animates correctly with bar visibility.

**How to apply:** Whenever you see a "BottomBar height + content padding" combination, ensure the height is reserved exactly once. Pattern works for action bars, snackbar zones, FAB clearance, sticky CTAs, persistent banners. Sentinel test: assert NO `Padding(bottom: ≥barHeight)` in the scrollable child's ancestors except the single `AnimatedPadding` wrapping the body switcher.
