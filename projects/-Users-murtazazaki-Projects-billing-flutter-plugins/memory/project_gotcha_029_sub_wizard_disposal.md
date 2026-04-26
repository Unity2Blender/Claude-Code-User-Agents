---
name: GOTCHA-029 Sub-Wizard Disposal Pattern
description: After await wizardStackProvider.push(), parent notifier may be disposed — requires context.mounted check + _disposed guard
type: project
---

GOTCHA-029: Notifier disposal after sub-wizard push.

**Why:** Crashlytics crash — `Cannot use the Ref of invoiceWizardProvider after it has been disposed`. `_handleAddItem()` awaits `wizardStackProvider.push()` (user interaction lasting seconds to minutes). During this async gap, the parent wizard's provider can be disposed (e.g., user system-back navigates away from WizardView). When the await completes, the captured `notifier` reference is stale.

**How to apply:**
- After every `wizardStackProvider.push()` await, add `if (!context.mounted) return;` before using the notifier (Layer 1: call site guard).
- In notifier methods called after async gaps (`addLineItem`, `clearCartIntent`, `setCartIntent`, `updateShipmentDetails`, `markSharedOrPrinted`), add `if (_disposed) return;` at the top (Layer 2: defense-in-depth).
- `registerDisposalGuard()` is a protected method extracted from `build()` — test subclasses call it to preserve real disposal wiring without running DB calls.
- Reference implementation: `_autoLaunchGstCalcItemEditor()` in `invoice_wizard_page.dart`.
- Regression test: `test/features/invoice_wizard/invoice_wizard_disposal_test.dart` (5 tests, tagged foundational + critical).
