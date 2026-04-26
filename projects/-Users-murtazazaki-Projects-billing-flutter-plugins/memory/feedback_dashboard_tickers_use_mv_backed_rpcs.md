---
name: Dashboard tickers use MV-backed RPCs
description: Visible dashboard tickers read aggregated values from existing MV-backed RPCs, never from client-side derivation on the items list
type: feedback
originSessionId: 596c93ec-b661-4171-b335-8e7c7ec8e56a
---
Visible dashboard ticker values (Stock Value, This Week, Out of Stock, Low Stock, To Collect, To Pay) MUST come from existing MV-backed aggregation RPCs:

- Bills/Parties → `get_dashboard_tickers()` RPC → `mv_dashboard_summary` (joined with live `v_stock_value` for stock value)
- Items Out of Stock / Low Stock → `get_reorder_alert_summary()` RPC → `mv_reorder_alert_summary`

Forbidden: client-side derivation like `inventoryDashboardProvider.where((s) => s.isLowStock).length` for ticker values; direct `.from('mv_*')` or `.from('v_stock_value')` SELECTs from Flutter; provider watches for tickers that aren't rendered.

**Why:** MVs are pre-aggregated source-of-truth, consistent across all surfaces (sidebar-list count + ticker count cannot diverge), and survive future schema changes that the existing RPCs already wrap. Client-side derivation duplicates aggregation logic and risks drift the user can spot when the sidebar count differs from the ticker. RLS REVOKE on `authenticated` already blocks direct MV access at the DB layer; the Flutter-side rule is belt-and-braces. Hidden/discontinued tickers are not rendered AND don't add provider watches — that distinction matters because watch-without-render still pays a network/cache cost.

**How to apply:** When designing any dashboard ticker (mobile or web), default to existing aggregation RPC paths. The sidebar list provider (`inventoryDashboardProvider`, `partyListProvider`) is for the LIST view, not for aggregate ticker values. Client-side `.where(...).length` on a list is only valid as a ticker source if no MV/RPC exists AND the operator has explicitly accepted the drift risk. Reverse direction: if a plan you're auditing prohibits an existing `*SummaryProvider` that wraps an aggregation RPC, challenge the prohibition — that provider may be the right answer. 2026-04-25 plan `virtual-meandering-hamming` v7 reversed v6's `.where(isLowStock).length` derivation after operator correction; canonical Items source is now `reorderAlertSummaryProvider`.
