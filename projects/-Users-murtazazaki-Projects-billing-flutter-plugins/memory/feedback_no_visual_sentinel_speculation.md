---
name: No speculative visual-sentinel fixes
description: Don't speculate that design tokens regressed; require an exact failing assertion file:line before touching `BillingSpacing` or similar visual sentinels.
type: feedback
originSessionId: 3825e5fa-e0a0-4ee5-8165-f00dc0651494
---
When a test summary mentions "tab banner spacing sentinels failing" or similar visual-token-related failures, do NOT preemptively edit design tokens (`lib/core/theme/billing_spacing.dart`, color tokens, density tokens) without an exact failing assertion at file:line.

**Why:** Visual sentinel files often already have asymmetric or intentional values (e.g., `BillingSpacing.tabBannerOuterMarginH` and `tabBannerOuterMarginV` at `lib/core/theme/billing_spacing.dart:17, :23, :28` already differ on purpose). Speculatively "restoring a regressed token" without seeing which assertion line in which test file is firing risks (a) inventing a regression that didn't happen, (b) breaking a tasteful asymmetry that production design intentionally chose, (c) wasting the slice's review budget on a non-bug.

**How to apply:**
- If a sentinel test is named in a failure list but the exact assertion isn't visible: defer the investigation to a ledger entry. Ship the analyzer + behavioral fixes first; revisit only if `make test-smoke` or `flutter test -t critical` surfaces a specific `expect(BillingSpacing.tabBanner*, …)` failure with file:line.
- Once a specific assertion fires, THEN: run `git log --oneline lib/core/theme/billing_spacing.dart` to find the recent change, decide intentional vs. regression with the user, and either revert the source change OR update the sentinel literal in lockstep.
- Quarantine via `@Tags(['flaky'])` with a 2-week deadline (`FLAKY-QUARANTINE-001`) is acceptable as last resort, but not a substitute for the file:line investigation.

**Source:** 2026-05-06 plan rejection. The original plan included "Restore `BillingSpacing.tabBannerOuterMarginH/V` token literal" as a critical-tier repair. The user's audit pointed out the tokens were already asymmetric at lines 17, 23, 28; speculating a regression risked breaking a working design. The fix should be: no-op unless an exact assertion surfaces.
