---
name: GOTCHA-037 DeferredInvoicePage Double Invalidation Blank Screen
description: billingContextProvider double-invalidation race causes blank screen when FirmWizard completes from DeferredInvoiceEntryPage flow
type: project
---

## GOTCHA-037: DeferredInvoicePage Double Invalidation Blank Screen

FirmWizardPage line 895 calls `ref.invalidate(billingContextProvider)` BEFORE popping. When DeferredInvoiceEntryPage resumes and calls `.refresh()` (which calls `ref.invalidateSelf()`), two invalidation cascades hit simultaneously.

**Why:** `PartyContactsTextField` watches `partySearchProvider(billingId)` which cascades from `billingContextProvider`. Two competing rebuild signals destabilize the widget tree, causing white scaffold with no content.

**Fix (2026-03-22):** Added `_isResuming` loading overlay in DeferredInvoiceEntryPage. `setState(() => _isResuming = true)` BEFORE `refresh()` removes `PartyContactsTextField` from the tree, preventing the cascade from hitting a live widget.

**How to apply:** Any page that calls `billingContextProvider.refresh()` while containing child widgets that watch downstream billing providers should show a loading state BEFORE the refresh to avoid the double-invalidation race.

**Key files:**
- `lib/billbook/onboarding/pages/firm_wizard/providers/firm_wizard_page.dart:895` — FirmWizard invalidation
- `lib/billbook/invoicing/pages/deferred_invoice_entry/deferred_invoice_entry_page.dart:125` — DeferredInvoicePage refresh
- `lib/state/providers/billing_context_provider.dart:429` — refresh() calls invalidateSelf()
