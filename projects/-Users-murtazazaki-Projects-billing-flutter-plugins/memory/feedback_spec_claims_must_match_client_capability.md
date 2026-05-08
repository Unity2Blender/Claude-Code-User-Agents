---
name: Spec claims about Worker/server inputs must verify Flutter/client sends them
description: Don't write spec text claiming Worker parses field X without grepping client code that X is actually sent. Adds plumbing as explicit migration step or removes the claim.
type: feedback
originSessionId: a077cb79-f425-4b9b-b98a-8852dbb40b6b
---
When writing a spec or migration plan that claims a Worker / RPC / backend reads a context field, verify the client actually sends it. The spec must EITHER (a) demonstrate the field is currently sent (grep client code), OR (b) add explicit plumbing tasks for client-side changes (Flutter widget reads, query-param construction, model wiring) BEFORE claiming the Worker parses it.

**Why:** 2026-05-05 banner placement plan claimed `locale` and `intent` would feed the Worker context hash, but Flutter only sent `placement, firm_count, is_signed_in, exclude_ids, build_number`. Worker plumbing without client plumbing is dead code; downstream tests assert behavior the live system can't produce.

**How to apply:**
- Before writing "Worker reads context fields A, B, C", grep client code (Flutter `BannerProvider`, `BannerService`, fetch URL builders) for each field.
- For fields not yet sent, list them as explicit client-side tasks under a "Flutter changes" section with file:line targets.
- Spec section ordering: client-changes BEFORE worker-changes, so reviewers see the full plumbing chain.
- Anti-starvation, ranking, cache-key shape — any spec claim that depends on a context field — gets the same scrutiny.
- Same rule applies to admin surfaces consuming Worker fields: don't claim admin shows metric M unless Worker emits M.
