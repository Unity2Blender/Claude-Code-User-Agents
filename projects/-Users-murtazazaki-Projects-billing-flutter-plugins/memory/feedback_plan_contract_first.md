---
name: Contract-first planning is scoped to risky cross-layer changes
description: File:line contract plans are mandatory for schema/API/auth/billing/cross-layer work, not for every small localized source or UI edit.
type: feedback
status: scoped-rewrite
---

**Rule:** Use contract-first file:line planning for changes that alter schema, migrations, RPC/API contracts, handlers, auth, billing, payments, data integrity, navigation contracts, or cross-layer behavior. Do not inflate small localized UI/source edits into exhaustive pseudocode-level plans.

**Why:** Contract-first planning caught real failures: stale SQL, missing file inventories, bad PostgreSQL patterns, duplicate providers, and non-atomic client flows. But applying the same weight to simple UI/source changes creates planning drag and encourages over-engineering.

**How to apply:**
- Mandatory file:line inventory for schema/API/handler/auth/billing/payment/destructive/drop/retire/cross-layer changes.
- Mandatory ship order when service contracts change; Flutter/client last when backend contracts move first.
- For small localized source or UI changes, verify the relevant files and proceed with a concise implementation note.
- If a plan starts adding runtime UI flags just to satisfy planning caution, remove them unless the user explicitly requested a flag.
- Before exiting plan mode on risky work, verify paths, signatures, column names, constraints, and function bodies against HEAD.
