---
name: Supabase sb_secret_* Key Format Change
description: New Supabase API keys use opaque sb_secret_* format (~42 chars) instead of JWT eyJ* (~180 chars) — validation must accept both
type: project
---

Supabase introduced new opaque API key format in mid-2025:
- **New format**: `sb_secret_<22-char-random>_<8-char-checksum>` (~42-45 chars)
- **Old format**: JWT `eyJ*` (~180+ chars)
- Both formats work with `@supabase/supabase-js` — no client changes needed
- Legacy key deletion planned for late 2026

**Why:** This caused a production 503 on 2026-04-08. The webhook-worker's `validateSupabaseEnv()` hardcoded `key.length < 100` which rejected the valid `sb_secret_*` key as "too short". The key was correct all along.

**How to apply:**
- NEVER use a fixed length threshold for Supabase key validation
- Use format-aware check: `key.startsWith('sb_secret_') ? key.length < 30 : key.length < 100`
- Fixed in: `shared/src/services/supabase.ts` and `webhook-worker/src/index.ts`
- All other workers (orders, catalog) should be checked for the same pattern
- Local `supabase start` does NOT support `sb_secret_*` keys yet (PostgREST expects JWTs locally)
