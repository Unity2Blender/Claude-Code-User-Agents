---
name: Place of Supply GST Rules
description: GST Place of Supply legal rules, Section 10(1)(ca) 2023 amendment, and implementation mapping
type: project
---

Place of Supply (PoS) feature implemented across Entry Step, notifier, InvoiceData, and 8 PDF templates.

**Key GST Sections:**
- Section 10(1)(a) IGST Act: Goods involving movement → PoS = delivery destination
- Section 10(1)(ca) IGST Act (Oct 2023 amendment): Unregistered buyer → PoS = address on invoice, else supplier's location
- Section 7(1) IGST Act: supplier state ≠ PoS state → inter-state → IGST
- Section 8(1) IGST Act: supplier state == PoS state → intra-state → CGST+SGST
- Rule 46(n) CGST Rules: PoS mandatory on inter-state Tax Invoices

**Implementation mapping:**
- `derivePlaceOfSupply()`: party GSTIN > party state > firm GSTIN fallback
- `isIntraState`: firmGstinStateCode == placeOfSupplyCode
- PoS stored as CHAR(2) state code in `invoices.place_of_supply` (DB migration pending)
- PDF renders "Place of Supply: StateName (Code)" below buyer details
- Row visible only when firm has GSTIN + party selected
- User can override via StatePickerSheet

**Why:** GST compliance gap — PoS determines IGST vs CGST/SGST and is legally required on inter-state invoices.

**How to apply:** When modifying tax calculation logic, always compare firm's GSTIN state code against placeOfSupplyCode. Never compare firm.state vs party.state directly (legacy fallback only for null PoS).
