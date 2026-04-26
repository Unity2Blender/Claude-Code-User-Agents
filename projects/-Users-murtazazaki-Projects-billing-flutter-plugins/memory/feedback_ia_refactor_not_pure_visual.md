---
name: Refactors that change data source / count semantics / routing are IA, not "pure visual"
description: If a plan changes which counts render on which tab, which providers feed them, or which taps route where — it's information-architecture, not chrome. Mislabeling invites scope regressions.
type: feedback
originSessionId: 6f570db4-b080-4d0e-ad2a-1cbeacc8c52b
---
When a plan involves:
- Moving a ticker from one tab to another (e.g., Stock Value Items→Bills)
- Changing the data source that feeds a ticker (e.g., from inventory aggregation to `DashboardStats.stockValue`)
- Changing the tap destination of a widget (e.g., Stock Value tap now routes to Items tab)

It is an **information-architecture refactor**, not a "pure visual" change. Operator correction during visual-overhaul plan 2026-04-24 — the v2 draft framed ticker reorg as "pure visual" and got this feedback:

> "Reclassify V2 as a dashboard information-architecture refactor, not pure visual. [...] Treat V2 as dashboard information architecture, not pure visual. Do not label billing-wide values as firm-scoped."

**Why the label matters:** "Pure visual" implies low blast-radius — testers + reviewers skip data-flow checks. But changing source/scope/routing can create real regressions (billing-wide values displayed with firm-scoped copy, broken lazy-loading contracts, tap routing to stale filters). Correct framing triggers the right gates: provider contract tests, lazy-loading tests, source-correctness tests, scope-labeling tests.

**How to apply:**
- Before framing a plan as "visual": audit whether it changes source, count semantics, or routing.
- If yes → label as **"Information-Architecture refactor"** (or `"visual + IA"`) in the plan title and scope line.
- Add preflight gates for data-scope verification (billing-wide vs firm-scoped, sync vs async, lazy vs eager).
- Add copy-review rules: billing-wide values must not use firm-scoped phrasing.

**Do not:**
- Do not label a ticker-reorg / count-swap / routing-change plan as "pure visual" — invites scope regressions.
- Do not assume copy can be neutral by default — if source is billing-wide, explicitly test that UI copy doesn't interpolate `currentFirm.name`.
