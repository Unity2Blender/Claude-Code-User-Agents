---
name: Run shipped-claim audit before building deferred work on prior PR claims
description: HEAD-FACT-PIN-001 verification via parallel grep + git log + symbol resolution catches phantom APIs in <2 min. Use before any spec write that builds on prior shipped work.
type: feedback
originSessionId: 88479857-ca35-4a9f-bd6b-ef7c16840ff0
---
**Rule:** Before composing a spec or amendment that builds on top of "previously shipped" work, run a SHIPPED-CLAIM-AUDIT verifying each claim at HEAD via parallel: (a) `git log --oneline -N` to see commits, (b) `grep -rn '<symbol>'` to verify symbols exist, (c) `find <path>` to verify files exist, (d) `wc -l <file>` to verify size matches expectation. Single Bash call with combined `echo "=== N. claim ===" && grep ...` blocks is the fastest pattern. Total time: <2 min for ~10 claims.

**Why:** On 2026-04-24, the brainstorming agent flagged `showRecordPaymentWizard(initialStep:)` as potentially phantom (claimed shipped but maybe not exposed in `payments.dart`). The shipped-claim audit verified it WAS shipped at `lib/navigation/entry_points/payment_entry_points.dart:71` with `initialStep` param at line 80, plus 5 other call sites. All 8 PR-β shipped claims (decode_failure_kind, banner_prewarm_failure_reason, readyV2, cold-start auth gate, cta_config caller-context, MATPU 9-arg, 000364, wizard initialStep, Worker validator, probe truth) verified in single audit. Catching even 1 phantom API early would have saved hours of cascading refactor on top of a non-existent foundation.

This pattern enforces HEAD-FACT-PIN-001 doctrine (added to ADR-010 §v8) without requiring a CI gate — relies on operator + reviewer discipline to invoke it before spec composition.

**How to apply:** Trigger conditions:
- About to compose spec/ADR amendment that lists "shipped" features as foundation
- About to refactor on top of prior PR's claimed work
- Audit agent flags any "this may not exist" concern
- Any plan v8.1+ amendment cycle (per the v7.1 lesson — that cycle missed `showRecordPaymentWizard` verification, leading to v8 having to catch it)

DON'T skip when:
- Prior session summary claims work shipped but you haven't read the actual files since
- Multiple agents disagree on what exists at HEAD
- About to write a spec that costs >1 hour to compose

Reference: ADR-010 §v8 Round 5 Q3 lock-in + Phase 3 in the §v8 "Markdown workflow example".
