---
name: Invoice allocation triggers — BEFORE vs AFTER, lock pattern
description: Verified facts about trg_fn_validate_invoice_allocation and trg_fn_payment_allocations_update_invoice
type: project
originSessionId: b5603ad8-f84c-4a63-bf3f-ede3326973a9
---
Verified at HEAD (post 000432, 2026-04-28):

- `trg_fn_validate_invoice_allocation` is a **BEFORE INSERT** trigger on `payment_allocations` (`triggers/invoicing.sql:993-1051`). Acquires its own `SELECT ... FOR UPDATE` on the invoice row and RAISEs `check_violation` (23514) when `(total_allocated + total_discount) > total_amount + round_off + 0.01`. Phase 0 (000432) reduced reliance on this trigger by adding a per-allocation FOR UPDATE inside the RPC body before INSERT — but the trigger remains as defense-in-depth for any non-RPC write path.
- `trg_fn_payment_allocations_update_invoice` is the **AFTER INSERT/UPDATE/DELETE** trigger that recomputes `invoices.amount_paid = SUM(allocated_amount + discount_amount)` for affected invoices (`triggers/invoicing.sql:791-825`). Locks invoice row FOR UPDATE during the recompute.

`save_payment_with_allocations` RPC concurrency notes:
- Edit mode pre-locks old-allocation invoice IDs sorted (`payments.sql` post-000432) before DELETE so the AFTER trigger's lock acquisition matches the new-allocation loop's `ORDER BY (value->>'invoice_id')::BIGINT`.
- Don't confuse the two triggers — BEFORE for validation, AFTER for invoice recompute.

`chk_payment_allocations_allocation_valid` CHECK constraint rejects `(allocated_amount=0, discount_amount=0)` rows. The Phase 0 skip-zero conditional only fires when a POSITIVE request was clamped to zero (e.g., already-fully-paid invoice). Explicit-zero callers fall through to the existing CHECK guard — preserved as caller-bug detection.

**How to apply:** When designing future migrations on this RPC or its triggers, remember the lock direction (RPC → invoices FOR UPDATE → trigger reentrant lock). Adding new triggers must respect ORDER BY invoice_id ASC global lock convention.
