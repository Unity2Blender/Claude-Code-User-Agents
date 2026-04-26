---
name: No anon-callable Supabase RPCs
description: Never create Supabase RPCs callable by the anon role — violates MATPU guard; all user sync must use service role key via Cloud Functions
type: feedback
originSessionId: cac4aa9c-2b1c-45cd-a852-10b796fb43d4
---
Never create Supabase RPCs granted to the `anon` PostgreSQL role for user-facing operations. All user sync operations must go through Cloud Functions using the service role key (`SUPABASE_SECRET_KEY` via `getSupabaseAdmin()`).

**Why:** The MATPU (Monthly Active Third-Party Users) defense is a 5-layer guard that ensures "Supabase is invisible to anonymous users." The transport-level chokepoint (`postgrestProvider`) throws `StateError` when `currentJwt` is null. Creating anon-callable RPCs undermines this defense and could create MATPU billing leaks. Even for signed-in users pre-claim, the correct pattern is server-side sync via CF, not client-side RPC bypass.

**How to apply:** When designing auth sync or user provisioning:
- Use Firebase Cloud Functions with service role key (server-side)
- Keep `ensure_user_exists()` (authenticated role) as housekeeping for legacy users
- Never add `GRANT EXECUTE ... TO anon` for user-facing functions
- The CF should be the sole path for creating/syncing user rows
