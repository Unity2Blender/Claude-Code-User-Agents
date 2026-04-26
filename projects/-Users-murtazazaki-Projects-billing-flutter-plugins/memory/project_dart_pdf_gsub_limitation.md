---
name: dart_pdf GSUB limitation — Hindi conjuncts broken
description: The dart_pdf package (DavBfr) cannot render Devanagari conjuncts; GSUB table support missing since 2020. Blocks perfect Hindi PDF rendering client-side.
type: project
originSessionId: b2f54b34-a08b-45b8-885b-ca81d548decb
---
The `pdf` package (pub.dev, by DavBfr) does NOT support OpenType GSUB (Glyph Substitution) tables. This is the mechanism required for Devanagari conjunct rendering (e.g., क्ष, त्र, ज्ञ — joined consonant forms).

**Impact**: Even with a Devanagari-capable font (Poppins, Mukta, Hind), Hindi conjuncts render as disconnected characters. Simple Hindi words without conjuncts render fine. Issue #198 has been open since 2020 with no fix.

**Why:** dart_pdf has its own pure-Dart TTF parser that reads glyph outlines but does not process GSUB/GPOS tables. Flutter's own `Text` widget renders Devanagari perfectly (uses Skia + HarfBuzz), but the `pdf` package does not leverage this.

**How to apply:** The only complete fix for Hindi PDF rendering is server-side generation using a tool with HarfBuzz (e.g., Chromium/Puppeteer on Cloud Run). Client-side PDF generation with dart_pdf will always have broken conjuncts. This is documented in Phase 2 of the font migration plan. When evaluating any future PDF library for Flutter, check GSUB support first.

**Key GitHub issues**: #198 (GSUB), #1518 (Hindi/Marathi), #1100 (non-English chars), #809 (complex glyphs).
