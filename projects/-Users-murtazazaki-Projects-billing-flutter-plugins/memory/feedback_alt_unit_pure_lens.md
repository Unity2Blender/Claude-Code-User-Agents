---
name: Alt Unit Pure Lens Model
description: Alternate unit toggle is a pure display lens, not a data mutation. Rate field read-only in alt mode. Base price captured from TextField at toggle time to preserve user edits.
type: feedback
---

Alternate unit toggle = **pure unit lens**, not state mutation.

**Rule:** The toggle converts quantity and derives rate from current primary price. It does NOT mutate the underlying pricing data.

**Why:** User may edit price (100→105) in primary mode before toggling. The derived alt price must use the edited price (105×12=1260), not the catalog price (100×12=1200).

**How to apply:**
- `_basePrimaryPrice = _unitPrice` captured from TextField at toggle time (not stored at init)
- Rate field `readOnly: _useAlternateUnit` — user must switch back to primary to edit
- Lock icon shown on rate field when in alt mode
- Toggle back restores `_basePrimaryPrice` to rate field
