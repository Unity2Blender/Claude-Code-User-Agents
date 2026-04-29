---
name: Two-pipe discount accounting model in save_payment_with_allocations
description: How declared global discount vs ad-hoc per-invoice write-off split is recorded; balance-trigger gap that 000434 follow-up closes
type: project
originSessionId: b5603ad8-f84c-4a63-bf3f-ede3326973a9
---
The payments domain has a two-pipe discount model:
1. **Declared global discount** — `payments.global_discount_amount`, set by user at top of payment wizard.
2. **Ad-hoc per-invoice write-off** — sum of `payment_allocations.discount_amount` minus the global portion.

Runtime split (in `save_payment_with_allocations`, payments.sql:577-590):
- `distributed_from_global = LEAST(total_discount, global_discount)`
- `adhoc_discount_amount   = GREATEST(total_discount - LEAST(...), 0)`
- Both written to `payment_audit_log.snapshot` for observability.

Phase 0 (000432, shipped 2026-04-28): F1 (`SUM(discount) <= global_discount`) DROPPED. Cross-invoice gate matches Flutter Fix C in `record_payment_state.dart:330-348` ("per-invoice over-allocation is the sole discount invariant").

**Accepted tech debt — closed by 000434:** ad-hoc write-offs reduce `invoices.amount_paid` (via `trg_fn_payment_allocations_update_invoice` summing allocated + discount) but DO NOT reduce `parties.outstanding_balance` / `party_fy_balances` because those balance triggers use `payments.amount + payments.global_discount_amount` — they don't see per-invoice ad-hoc reductions. Live impact zero today (pre-000432 F1 rejected ad-hoc entirely; no production rows with ad-hoc-only). Queued follow-up: `000434_adhoc_writeoff_balance_accounting` adds column + updates balance triggers.

**Why:** When designing payment-related migrations, remember party/FY balance is computed from `payments.amount + global_discount_amount` directly, NOT from a SUM over `payment_allocations`. Don't assume the trigger will catch ad-hoc reductions — it won't until 000434 ships.

**How to apply:** When working on payment_save_pipeline in Flutter, do NOT aggregate per-invoice discounts into `payment.global_discount_amount` — that destroys the audit-distinct split. Send `state.globalDiscount` as-is; the server reconstructs the split via the runtime LEAST/GREATEST formula.
