---
name: KV purge uses one-shot target, never hand-rolled two-step
description: For banner/placement-config cutovers, always use `make admin-purge-banner-all ENV=production` — do NOT purge `banner:v3:*` and `placement-config:*` as separate admin-purge calls
type: feedback
originSessionId: e2f8e44b-274b-414f-a32e-6a5f6b0e99b4
---
**Rule:** When purging KV cache for a banner-related cutover, invoke `make admin-purge-banner-all ENV=production` (single target) rather than two separate `make admin-purge PATTERN=banner:v3:* ENV=production` + `make admin-purge PATTERN='placement-config:*' ENV=production` calls.

**Why:** The tutorials Worker has a schema-cache invariant (documented in `microservices/tutorials/CLAUDE.md` under "RPC Criticality Contract"):

> `placement-config:{placement}` KV TTL (1h) MUST be > `banner:v3:{placement}` KV TTL (30m) so config mutations age out banner pools naturally.

If an operator purges `banner:v3:*` FIRST (e.g. by running the two admin-purge calls in the order the plan listed them), the Worker immediately rebuilds the banner cache using whatever `placement-config:*` state is still cached. That key hashes in the cat/intent fingerprint from the stale placement config — so the rebuilt `banner:v3:{placement}:{build}:{staleCatHash}:{staleIntentHash}` lives alongside future `…:{freshCatHash}:{freshIntentHash}` keys for up to 30 minutes, fragmenting the cache and producing inconsistent banner pools per user.

`make admin-purge-banner-all` handles the ordering correctly (placement-config first, THEN banner:v3:*), closing the race window. Makefile is the source of truth; audit-proposed sequences should always defer to it.

**How to apply:** In any runbook, cutover plan, or operator procedure that touches banner/placement config, reference `make admin-purge-banner-all ENV=production` explicitly. Flag draft plans that hand-roll the two-step sequence — even if the listed order looks right, the one-shot is the canonical path + dodges the trap.

**Out of scope for the one-shot:** `playlist:*` and `categories` caches are separate namespaces unaffected by the banner invariant — purge them as distinct admin-purge calls when the cutover touches playlist RPC behavior. Example from the 2026-04-22 steady-boot plan:

```bash
make admin-purge-banner-all ENV=production          # one-shot: banner:v3:* + placement-config:*
make admin-purge PATTERN='playlist:*' ENV=production
make admin-purge PATTERN='categories' ENV=production
```
