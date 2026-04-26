---
name: Readiness is authored product state, not URL convention
description: Never derive domain readiness flags (is_media_ready, is_published, etc.) as GENERATED columns from string/URL patterns; that couples state to naming conventions and violates the separation between lifecycle and derived attributes
type: feedback
originSessionId: ae1c4a54-b698-4ebb-a33e-67a6d5fbe956
---
**Rule:** For domain readiness flags (`is_media_ready`, `is_published`, `is_live`, etc.), always use an authored BOOLEAN (or enum like `media_status placeholder|ready|archived`) with an optional validator trigger that WARNS (not BLOCKs) on suspicious combinations. Never model readiness as a GENERATED STORED column derived from URL/filename patterns.

**Why:** Generated-from-convention flags couple product state to CDN naming rules — changing the placeholder URL format silently flips the flag, and no migration can authoritatively "mark this reel ready." Readiness must stay separate from both publication_status and URL shape. The user explicitly called out that my 000303 generated-column proposal for `tutorials.is_media_ready` was the highest-leverage flaw in the plan because it "turns readiness into a CDN naming convention, not a product state."

**How to apply:** When modeling lifecycle/readiness in Postgres, the column owns the state. A generated "helper" derived from URL patterns is advisory only — never the source of truth. Applies to any future `is_*_ready`, `is_live`, `is_approved` columns.
