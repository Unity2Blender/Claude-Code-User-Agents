---
name: Payment allocation must not hard-clamp to outstanding
description: Per-invoice allocation CurrencyField must NOT clamp server-side; preserve "Over by ₹X" UX, disabled-save state, and analytics events.
type: feedback
originSessionId: 73bde94f-3069-443c-8d3a-72e11c7d77d7
---
P0 HARD — Do not propose hard-clamping in `RecordPaymentNotifier.setAllocation` or in any per-invoice allocation `CurrencyField`. Preserve the existing "Over by ₹X" UI affordance, the disabled-save behavior in over-allocated state, and the analytics events that fire on over-state transitions.

**Why:** On 2026-05-03 the user explicitly rejected TODO #4 ("CurrencyField clamp focus-sync") because adding a hard-clamp would silently break three existing contracts:
1. The "Over by ₹X" UI affordance (users rely on this to spot allocation mistakes).
2. Disabled-save behavior in the over-allocated state (prevents accidental over-payment commits).
3. Analytics events that fire on over-state transitions (lost telemetry → blind spot in payment-flow funnels).

The TODO's loose phrasing ("syncs even when focused") inverted the actual code state — the focus-guard at `lib/billbook/common/components/currency_field.dart:133-168` already blocks all external syncs while focused. The user reviewed the proposal to add a narrow `acceptExternalClamp` opt-in prop and rejected it as a regression for the over-allocation UX surface.

**How to apply:** When a future spec proposes "clamp the per-invoice allocation to outstanding balance" or "auto-correct over-allocation server-side", treat as P1 reject. Require new UX research that explicitly re-litigates:
- The Over-by-₹X display semantics.
- The disabled-save contract.
- The analytics-event firing pattern on over-state transitions.

Reopen requires new evidence (UX study, support ticket pattern, or accessibility audit) — not just "wouldn't this be cleaner?". Memorialized in `lib/billbook/payments/pages/record_payment_wizard/CLAUDE.md` (ADR-2026-05-03).

The broader CurrencyField audit (cursor jump, IME, RTL, accessibility, paste handling) is INDEPENDENT of clamp and tracked separately as TODO #7 in the deferred-spec backlog.
