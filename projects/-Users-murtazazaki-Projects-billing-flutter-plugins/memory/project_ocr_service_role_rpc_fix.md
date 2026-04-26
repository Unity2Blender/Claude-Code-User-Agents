---
name: OCR Service Role RPC Fix (P0)
description: bill-ocr entity matching RPCs failed because service role has no JWT sub claim — fixed with _internal variants
type: project
---

Entity matching in bill-ocr microservice was silently failing for ALL OCR scans. Root cause: `search_parties()` and `search_items_by_name()` RPCs have `has_billing_access()` guards that call `firebase_uid()` which reads `request.jwt.claims->>'sub'` — NULL for service role key.

**Fix (migration 000180)**: Created `search_parties_internal` and `search_items_by_name_internal` — exact copies of hardened bodies (H-02 ILIKE escaping, `deleted_at IS NULL`) without `has_billing_access()` guard. REVOKEd from PUBLIC/authenticated/anon, GRANTed only to service_role.

**Why:** Microservice async workers (Cloud Tasks) don't have user JWTs — they run with service role. Follows the established `claim_ocr_scan` (000096) and `refresh_ocr_candidates` (000173) pattern.

**How to apply:** When creating new RPCs called by microservices, always create `_internal` variants without JWT-dependent guards. Never add `role = 'service_role'` bypass to the same function — mixing trust levels in one function violates audit clarity.
