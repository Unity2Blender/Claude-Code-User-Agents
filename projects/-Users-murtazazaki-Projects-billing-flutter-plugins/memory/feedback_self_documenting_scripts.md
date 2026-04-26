---
name: Self-documenting scripts (runbook header)
description: Operational scripts must carry a runbook-style header comment, not just a usage hint
type: feedback
originSessionId: f0d434da-97f9-4d12-bc47-465747837ead
---
For one-off operational scripts (backfills, reconciliations, damage-control tools) that the user expects to re-run "time to time", the header comment must read as a runbook, not just a one-line usage hint.

**Why:** Explicit user ask on 2026-04-22 while planning the Firebase→Supabase backfill enhancement: *"this should also be documented within this script as well because this is something which I can use time to time."* The April 2026 generator (`9026713a`) had only a 4-line usage comment — too thin to confidently re-run a year later without re-reading the codebase to remember why it exists.

**How to apply:** When creating or editing scripts in `scripts/` or `microservices/*/scripts/`, structure the header as:

```
# ============================================================================
# <script-name>
# ============================================================================
#
# ## What this does       ← 2-3 sentences, what the script's deliverable is
# ## When to use          ← 2-4 numbered scenarios, each tied to a real incident
# ## Usage                ← full flag matrix with concrete examples
# ## Idempotence          ← what re-runs do
# ## End-to-end runbook   ← shell commands a future-self can copy-paste
# ## Schema/context notes ← surprising constraints (e.g., FK directions)
# ## Related migrations   ← migration numbers + 1-line purpose
# ## Related commits      ← SHAs + 1-line purpose
# ## Why this exists      ← what canonical path was rejected and why
# ============================================================================
```

Make `--help` print the full block (not just the title): use `awk 'NR==1 && /^#!/ {next} /^#/ {sub(/^# ?/, ""); print; next} {exit}' "$0"` rather than a hand-tuned head/tail count.

This rule applies to operational/runbook scripts. App code, build scripts, and one-shot debug snippets don't need this treatment.
