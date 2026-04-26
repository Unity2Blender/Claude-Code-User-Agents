---
name: Smart font selection — detect script, don't blanket-swap
description: User prefers conditional font selection (English→Roboto, non-Latin→Poppins) over replacing Roboto globally. Minimize visual regression for English users.
type: feedback
originSessionId: b2f54b34-a08b-45b8-885b-ca81d548decb
---
When adding multi-script font support, do NOT blanket-replace the existing font. Instead, use a smart helper function that detects whether text is English/Latin and conditionally selects the font.

**Why:** Replacing Roboto with Poppins globally changes the visual appearance of ALL invoices, even English-only ones. This creates unnecessary visual regression for the majority of users. The correct approach is zero change for existing users, new capability only where needed.

**How to apply:**
- Build a `isLatinOnly(String)` helper (or similar binary check) — NOT per-script enum detection
- English/Latin text → use default font (Roboto) — no visual change
- Non-Latin text (Hindi, Devanagari, etc.) → use Poppins
- Keep it simple and fast — avoid calculation-intensive per-character analysis
- Design via TDD — test the helper function independently with unit tests
