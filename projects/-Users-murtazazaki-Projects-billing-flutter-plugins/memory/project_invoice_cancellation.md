---
name: Invoice Cancellation Feature
description: Invoice cancellation feature plan — Sales+Purchase only, bottom sheet UX, 2 concurrency bugs found, 9-phase TDD implementation
type: project
---

Invoice cancellation feature in active development. Plan: `.claude/plans/distributed-tinkering-simon.md`

**Scope:** Sales + Purchase invoice cancellation only. No PDF watermark (v2). No CN auto-generation.

**Key decisions:**
- Auto-cascade payment handling (existing triggers handle it)
- 2 new columns: `cancellation_reason TEXT` + `cancelled_at TIMESTAMPTZ`
- Bottom sheet (not dialog) with RadioListTile reasons, safe-default "No, Keep" button
- Block cancellation when active CN/DNs reference the invoice
- Full detail + red banner for cancelled invoice review

**Critical bugs found during audit (Phase 0):**
- **RC-CANCEL-PAY [CRITICAL]**: `trg_fn_validate_invoice_allocation` doesn't check `is_cancelled` — payment can be allocated to cancelled invoice in race condition
- **GOTCHA-C-WAC [HIGH]**: Cancel stock reversal doesn't update `avg_cost_per_unit` — WAC drifts

**Why:** GST compliance (Companies Accounts Rules 2014), competitive parity with myBillBook, audit trail for CAs.

**How to apply:** When touching invoice triggers, allocation triggers, or stock reversal logic, verify these bugs are fixed. The cancel_invoice() RPC is the canonical entry point for cancellation (not raw UPDATE).
