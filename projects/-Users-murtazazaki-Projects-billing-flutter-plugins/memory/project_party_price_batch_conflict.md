---
name: Party-price × Batch-tracking Conflict
description: MBB's app-wide mutual exclusivity is an SMB-mobile outlier; Zoho/Tally/Marg/SAP/NetSuite all permit both. Decision needed on per-item vs firm-level vs emergent-forking policy.
type: project
originSessionId: 052b43bd-aa85-4e73-9454-ba3989787521
---
## Discovered 2026-04-14 during mutable-hatching-star plan

MyBillBook forces **app-wide mutual exclusivity**: a firm must disable batching to enable party-wise pricing, or vice versa. Stated in their KB article https://knowledge.mybillbook.in/en/articles/3297679-how-to-use-party-wise-item-prices — "Batching must be disabled to use this feature."

## Why MBB did it (inferred, not confirmed)
1. SMB-mobile UX constraint: tap-and-bill flow can't reconcile 3-way pricing (batch cost + batch MRP + party price) in a single-rate line-item input.
2. Data-model choice disguised as feature: 2-way `(party_id, item_id)` table is cheaper than 3-way `(party_id, item_id, batch_id)` override table.
3. No GST/IRP mandate forces this — it's purely product-design.

## Every ERP allows both
- **Tally Prime**: Price Lists + Price Levels per party ledger compose with batches+expiry (confirmed in https://tallysolutions.com/tally/price-lists-and-price-levels-in-tally-prime/)
- **Zoho Inventory**: Batch is per-item opt-in; Price List is per-customer; invoice stores selected batch + applied rate independently
- **Marg ERP (pharma)**: Rate1/2/3/4 + party-rate + batch-rate all coexist — explicitly designed for pharma where this is non-negotiable
- **SAP B1 / NetSuite**: Batch = cost-bearer, customer = price-bearer, meet at invoice line with specific-cost COGS
- **BUSY 21**: Markets "date-wise or party-wise stock tracking with automatic batch number generation" as a composed feature

## Semantic pattern across ERPs
- Batch = **cost** identity (purchase_rate, mfg_date, expiry, MRP ceiling for pharma)
- Customer/Party = **price** identity (negotiated sell rate)
- They meet at the invoice line: `margin = sell_rate − batch_cost`, computed per line, stored denormalized for reports
- Margin legitimately varies per batch per customer — that's a feature, not a bug

## User's stated intent (project-specific)
- NO firm-level preference (reject MBB's approach)
- Item-level detection: if an item has party prices for >1 party in a financial year (active distinct prices), item is considered **forked**
- Forked items allow BOTH party-pricing AND batching simultaneously with drift acknowledgment
- Emergent forking, not policy-driven

## How to apply
When designing/implementing item-level policy for billing_flutter_plugins:
1. Reject MBB's app-wide toggle as restrictive outlier
2. Reference Zoho/Tally/Marg precedents for composed-feature UX
3. Detect forking automatically from usage (first party-price + first batch = forked state)
4. Use batch cost at allocation time for margin truth; accept drift; surface via reports
5. Never silently restrict — always warn and allow (Tally's permissive posture)

## Critical plan bugs discovered during this audit (to fix)
1. Plan v1.0 D6/D18/SCHEMA-G-001 reference `save_invoice_atomic` — **actual RPC is `create_and_finalize_invoice`** at `supabase/schemas/functions/invoicing.sql:301-553`
2. HC-1 ordering was wrong — batch consumption fires via `trg_fn_update_stock_from_movement` on stock_movements insert BEFORE Phase C banner can run. Correct order: `insert lines → consume batches (trigger) → C-validate → finalize → G-upsert (if allowed)`
3. Phase G silent write needs a "mode-aware no-op" when item is batch-tracked

## Ultraplan findings (2026-04-14, post v1.2)
1. **`pricing_mode` must be DERIVED, not stored.** Storing on `items` drifts on delete/depletion/restore/backfill. Use `effective_pricing_mode(item_id)` helper function mirroring existing `item_tracking_type()` at `supabase/schemas/functions/inventory.sql:1524-1565`. Base derivation on live facts: `item_party_prices` + `stock_batches`.
2. **OCR routing bug:** `OcrReviewNotifier.createItem()` at `lib/billbook/ocr/pages/ocr_review/state/notifiers/ocr_review_notifier.dart:483-497` writes OCR price only to `default_net_price`. Quick-create sheet exposes only one price field. Also writes `item_code` which has no SQL backing — phantom field write.
3. **Barcode lookup functions missing:** `search_item_by_barcode()` / `search_variant_by_barcode()` exist only in `supabase/backups/schema_20260125.sql:5169-5456`, not in active `supabase/schemas/**/*.sql`. Phase D split into D.1 (Lookup/Schema restoration + data migration) and D.2 (Scanner UI).
4. **Phase F file paths stale:** `inventory_list` not `item_list`. Filter enum at `lib/billbook/inventory/pages/inventory_list/state/models/inventory_filter.dart:4-15` currently only has `all, lowStock, outOfStock, trashed`. Add `expiringSoon, expired, forked` enum values + view/count support before chip UI.
5. **D5 removal of `effectivePrice` is multi-file migration** — `item_search_result.dart:96-144`, `definition_basic_step.dart:564-579`, `item_search_notifier_test.dart:236-260`, `item_search_result_vendor_test.dart:47`.
6. **Batch cost revaluation scope clarified:** `adjust_batch_cost()` revalues REMAINING stock only, not historical sold lines. Sold-line COGS + margins stay frozen (append-only principle).
7. **Web wizard not truly sectionized** — `item_editor_wizard_web.dart:182-190` still mounts whole mobile steps inside sidebar sections. Cognitive-load problem if Phase B/C are to land on web; needs separate plan.
