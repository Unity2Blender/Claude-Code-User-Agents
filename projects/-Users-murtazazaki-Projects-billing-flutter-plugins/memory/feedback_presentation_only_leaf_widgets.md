---
name: Presentation-only leaf widgets — no ref.watch in tickers/cards
description: Leaf widgets (tickers, ticker-like summary cards) must be pure presentation. Parents own provider reads and pass resolved values down.
type: feedback
originSessionId: 6f570db4-b080-4d0e-ad2a-1cbeacc8c52b
---
Leaf widgets rendering dashboard metrics (tickers, summary cards, stats tiles) must extend `StatelessWidget`, **not** `ConsumerWidget`. Provider reads (`ref.watch`, `ref.listen`) live in the parent dashboard widget (`dashboard_mobile.dart` or equivalent). The leaf receives resolved `label` / `value` / `subtitle` / `icon` / `colorResolver` / `onTap` via constructor.

**Why:** Operator correction during the visual-overhaul plan 2026-04-24. My v2 draft made each ticker subclass `ConsumerWidget` that watched its own provider. Operator rejected: (1) breaks lazy-loading contracts — every ticker instantiation eagerly attaches a listener even if the tab isn't visited; (2) couples presentation to data source; (3) makes widget tests hard — each test needs a `ProviderScope` + override even for pure styling assertions; (4) reverses V11 Riverpod architecture principle that non-page widgets should not own provider reads.

**How to apply:**
- If designing a dashboard ticker / summary card / stats tile → `extends StatelessWidget`. Constructor takes resolved data.
- If redesigning existing `ConsumerWidget` leaves, migrate to presentation-only and hoist `ref.watch` to the closest page/view-level widget.
- Parent uses `ref.listen` or `ref.watch` and passes plain data into children. Parent also owns the `onTap` callbacks — leaves invoke, don't resolve.
- Exception: true "feature containers" (pages, sheets, wizard steps) keep `ConsumerWidget` because they are the unit that starts a feature's data lifecycle.

**Test discipline:**
- Add a static grep sentinel under the leaf directory (`lib/dashboard/widgets/tickers/`) asserting zero `ref.watch` imports.
- Leaf widget tests use `WidgetTester.pumpWidget(MaterialApp(home: StockValueTicker(stockValue: 5_24_000, onTap: callback)))` — no ProviderScope needed for styling assertions.

**Do not:**
- Do not justify `ConsumerWidget` leaves via "test ergonomics" (`find.byType(StockValueTicker)` works the same with either base).
- Do not mix presentation-only ticker subclasses under a `ConsumerWidget` base.
