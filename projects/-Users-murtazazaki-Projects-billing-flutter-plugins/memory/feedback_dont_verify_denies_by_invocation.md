---
name: Don't verify permission deny rules by invoking them
description: Presence in settings.json is the contract — never include "attempt the denied command" as a verification step
type: feedback
originSessionId: d7a6b2d2-488d-4de6-8473-29173540f834
---
Do not include "attempt the denied command and confirm it blocks" as a verification step in plans. Presence of the rule in `.claude/settings.json` (or `settings.local.json`) is sufficient evidence that the harness will block it. The verification section for permission hardening should stop at "rule is present in the right file".

**Why:** During the 2026-04-22 Supabase migration down post-mortem plan review, the user removed a verification step that said *"attempt `supabase migration down --last 1 --linked` via Bash tool — harness blocks with 'denied by settings'. Repeat for `db reset --linked`, etc."* Their reasoning: invoking destructive commands just to watch them get denied is unnecessary theatre, it risks surprising the harness or the user, and the guarantee we actually care about — the rule being in the config file — is already satisfied by a simple grep/jq check. The harness enforcement layer isn't under test; our config content is.

**How to apply:** When writing verification sections for plans that add deny rules, cap the check at "rule appears in `settings.json` with correct glob". Do NOT add steps like "try running the blocked command to confirm". Applies to any command, destructive or not. If a user explicitly asks to test that the deny rule works end-to-end, fine — but never propose it unprompted.
