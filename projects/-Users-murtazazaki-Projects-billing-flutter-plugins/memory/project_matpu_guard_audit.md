---
name: MATPU Guard Audit Results (2026-04-03)
description: 5-layer anonymous user defense — audit findings, fixes applied, and architecture decisions
type: project
---

MATPU guard hardened to 5-layer defense-in-depth (commit d5e0993a).

**Why:** ~70K MAU on Supabase billing with only 7-8K signups. Even denied requests count as MAU activity. User's principle: "anonymous users should not even be aware about making any request to Supabase."

**How to apply:** Any new Supabase call path must go through one of these layers. When reviewing code, verify no direct `postgrestProvider` access bypasses auth checks.

**5 Layers (client → server):**
1. Layer 1: `matpuSafeJwtProvider` (supabase_rest_client.dart:230) — returns null JWT for anon
2. Layer 1.5: `postgrestProvider` (supabase_provider.dart) — throws StateError if JWT null
3. Layer 2: `billingContextProvider` — returns null for isAnonymous
4. Layer 2.5: `canPerformSupabaseRpc()` (anonymous_rpc_guard.dart) — mirrors Layer 1
5. Layer 3+: Server-side RLS + ensure_billing_exists() + Cloud Functions

**Key decisions:**
- Layer 1 and 2.5 intentionally duplicate same logic (defense-in-depth, NOT accidental)
- `postgrestProvider` throws StateError (not returns null) — forces callers to handle
- Reference tables locked to `TO authenticated` server-side (migration 000218)
- Surgical function revoke (not blanket) — RLS helpers stay accessible to anon role
- Sentinel tests rewritten with mocktail to test PRODUCTION code directly

**Critical files (changes here require MATPU audit):**
- `lib/core/services/supabase_rest_client.dart` (Layer 1 guard)
- `lib/state/providers/supabase_provider.dart` (Layer 1.5 guard)
- `lib/core/services/anonymous_rpc_guard.dart` (Layer 2.5 guard)
- `apps/gst_calculator/lib/app/deferred_init.dart` (GST Calculator init)
