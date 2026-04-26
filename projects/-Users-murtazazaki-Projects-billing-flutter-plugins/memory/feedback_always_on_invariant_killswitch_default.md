---
name: Always-on invariants ship direct; UI killswitches require explicit request
description: Product-critical invariants should default to the correct behavior. Do not add UI killswitches or default-off UI flags unless the user explicitly asks.
type: feedback
status: scoped-rewrite
originSessionId: 98cb53ff-0797-4ca2-a60d-442bd1a6f79f
---

**Rule:** If a behavior is a product invariant, implement the invariant directly. Do not ship it default-off behind a flag, and do not add a UI killswitch unless the user explicitly asks for one.

**Why:** Default-off for an invariant keeps the water off and hides leaks. But the old memory overcorrected by normalizing UI killswitches as emergency levers. Current doctrine is: UI changes are client-side and should ship directly. Safety mechanisms belong where the real risk lives, usually backend/data-plane or billing logic, not in speculative UI branches.

**How to apply:**
- For UI invariants, render the correct UI directly and add tests/fallbacks where useful.
- For backend/data-plane invariants with production blast radius, add the smallest safety mechanism that protects users and can be justified by a distinct failure mode.
- If a spec says `*_enabled=false` for an invariant, reject the default-off posture.
- If a spec adds a UI killswitch without an explicit user request, remove it.
- Pair always-on behavior with targeted evidence such as structured logs, counters, probes, or tests when the blast radius warrants it.

**Superseded history:** The original Banner v4 memory preserved KV killswitches for GST banner invariants. That remains historical context for that shipped system only; it must not be generalized into new UI flag doctrine.
