---
name: Invoice ID allocator reclaim semantics
description: get_next_available_invoice_id reclaims same-FY tombstones first; collision scope is (firm, mode, FY, number); prefix is informational only.
type: project
originSessionId: 73bde94f-3069-443c-8d3a-72e11c7d77d7
---
`get_next_available_invoice_id` reclaims same-FY tombstones first, then advances `invoice_mode_prefs.last_used_id + 1`. Slot occupancy scope is `(firm_id, invoice_mode, fin_year, invoice_number)`; `invoice_prefix` is informational and does NOT participate in collision detection.

**Why:** This shapes the `check_invoice_id_available` RPC (Phase 1.A spec, 2026-05-03) status taxonomy: a tombstoned slot is `reclaimable_current` if it equals the allocator's next pick, otherwise `tombstone_blocked`. The RPC must distinguish allocator-visible reclaim (UI shows "Will reclaim on save", Save enabled) from skipped tombstones (UI shows "Permanently retired", Save disabled). Failing this distinction forces users to manually iterate counters and creates surprise renumbering at save time.

The shared helper `_next_invoice_number_pick(billing_id, firm_id, canonical_invoice_mode, fin_year)` should encapsulate this logic so that the new check RPC and the existing `get_next_available_invoice_id` cannot diverge.

**Authoritative HEAD facts (verified 2026-05-03):**
- `fn_invoice_slot_occupied(billing_id UUID, firm_id BIGINT, invoice_mode TEXT, fin_year INTEGER, invoice_number BIGINT) RETURNS BOOLEAN` at `supabase/migrations/000409_baseline_v3.sql:6730-6750`. Consults BOTH `invoices` (with `is_draft=FALSE AND is_cancelled=FALSE` and FY scope via `get_invoice_fy_start_year`) AND `invoice_tombstones`.
- `invoices` columns at `000409_baseline_v3.sql:9070-9135`: `invoice_prefix TEXT NULL`, `invoice_number BIGINT NOT NULL`, CHECK on `invoice_mode` IN `('PROFORMA','SALES','CREDIT_NOTE','DEBIT_NOTE','PURCHASE','ESTIMATE','QUOTATION','PURCHASE_ORDER','DELIVERY_CHALLAN')`.
- `canonicalize_invoice_mode(p_input)` at `000409_baseline_v3.sql:1985` — Dart-style aliases (e.g. `salesInv`) → canonical UPPER_SNAKE.
- `validate_invoice_number` already blocks live finalized + tombstone collisions; `insert_invoice` retry loop is the save-time authority.

**How to apply:** Any future invoice-numbering UX, audit, or migration that touches collision/allocation MUST distinguish allocator-visible reclaim from skipped tombstones. Prefix arguments are for display, not collision. Suggested-number values returned by any read RPC MUST match the live allocator's pick (extract a shared helper, do not re-derive).
