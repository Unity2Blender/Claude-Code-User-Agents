---
name: CTA Audit Follow-up Items
description: 3 open items from 6-agent audit of CTA navigation wiring (2026-04-09) — version pre-check gap, nav stack pop-awareness, gate permutation tests
type: project
originSessionId: 29ba6916-0dc5-4620-8e42-d5ad710f2971
---
From the 6-agent audit of CTA Feature Target Registry (2026-04-09). Implementation complete but 3 items remain:

**1. Version gate pre-check gap (Architecture Finding 6)**
`resolveCtaAction()` never compares `currentBuildNumber()` against `TutorialVideo.minBuildNumber`. The Worker's `get_video_playlist()` RPC filters by build number server-side, but if a video makes it through (e.g., cached), there's no client-side pre-check. Add a version comparison in `ReelCtaButton` or `reel_video_card.dart` before calling `resolveCtaAction()`.

**Why:** Belt-and-suspenders. Low risk since Worker already filters, but completes the defense.

**How to apply:** Add before `resolveCtaAction()` call: `if (video.minBuildNumber != null && callbacks.currentBuildNumber() < video.minBuildNumber) { callbacks.onUpdateApp(); return; }`

**2. Navigation stack pop-awareness (Architecture Finding 5)**
After wizard completes ON TOP of Reels (fullscreenDialog), CalcHome's domain-aware tab transition (`BillingEntryOutcome.enteredBillingUi` → switch to Bills tab) doesn't fire because Reels absorbs the pop. Currently user returns to Reels (per locked decision). Minor UX gap.

**3. Gate permutation test file**
Test audit recommended `cta_gate_permutation_test.dart` (~16 tests) for auth×entitlement×version matrix and async boundary failures. Would strengthen widget-level flow test coverage.
