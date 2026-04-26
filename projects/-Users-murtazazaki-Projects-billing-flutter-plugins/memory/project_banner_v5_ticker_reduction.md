---
name: StatsGrid variants + filter-aware Sales provider (L59/L103)
description: Home 4-card → 2-card reduction with filter-aware Sales; Parties gets a new 2-card aggregate row
type: project
originSessionId: 519b0f7a-5681-477f-81a4-0ccaf3994e4a
---
Banner v5 CP-12 and CP-19 introduced `StatsGridVariant {legacyFour, home, parties}` so the dashboard can swap KPI compositions per tab without duplicating the widget.

**Why:** L59 called for 4→2 ticker reduction on Home to make room for the banner sliver; L103 specified a filter-aware Sales KPI (static "Sales" label, value computed from current filter range). Phase D added a parties aggregate row (ToCollect + ToPay summed across parties).

**How to apply:**
- Default `variant: StatsGridVariant.legacyFour` preserves the pre-CP-12 4-card layout for callers that don't opt in.
- Home 2-card row: `variant: home` uses `stats.toCollect` + `stats.thisWeekSales` as the "Sales" slot. Parent wires `salesAmountForFilterProvider` into `stats.thisWeekSales`.
- Parties 2-card row: `variant: parties` uses `stats.toCollect` + `stats.toPay`. Parent wires `partiesAggregateProvider` (sums RECEIVABLE/PAYABLE balance_status).
- Label stays static "SALES" / "TO COLLECT" / "TO PAY" — the visible filter chip (or tab context) is the dynamic scope, not the label (AP3 simplification).
- Sales sum is `sale - salesReturn` — sales-return refunds deduct from gross.
- Parties aggregate treats SETTLED / null status as zero (no contribution).
- Negative balances use `abs()` defensively (PAYABLE stored as +ve in production).
