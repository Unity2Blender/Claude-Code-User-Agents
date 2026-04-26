---
name: Rate Limiting Architecture Patterns
description: Lessons from OCR rate limiting audit — KV-only is weak, day bucketing matters, upgradeTo() wrong for recovery
type: feedback
---

**KV-only throttling is too weak for abuse-resistant enforcement.** Always use hybrid KV fast-path + append-only Postgres audit table as authoritative ledger. KV counters are fire-and-forget (can lose increments), have no audit trail, and lack durability guarantees.

**Why:** KV `waitUntil` counter increments are fire-and-forget. If the increment fails silently, the next request passes the limit. For monetization-critical paths (scan quotas, feature gates), the DB must be the source of truth.

**How to apply:** Any rate limiter in this project should follow the pattern: KV for speed (~1ms), Postgres for authority (~50ms fallback). Record admission decisions in an append-only table.

---

**Day bucketing: use admission day, not completion day, for async workflows.** When a scan is submitted at 23:58 IST but completes at 00:02 IST, `quota_counted_at = NOW()` at completion records the wrong day. The audit row at admission time captures the correct IST submission day.

**Why:** Pre-midnight submit + post-midnight completion = miscounted quota if using completion timestamp.

**How to apply:** Any async pipeline with daily limits should record the admission day at submission time, not at processing completion.

---

**`upgradeTo()` is wrong for recovery flows.** It has a session cap, returns only `bool`, and optimistically mutates tier state. For paywall recovery (user denied → upgrade → retry), use `showPaywallCallbackProvider` directly. This gives the page control over the PaywallWizard lifecycle and doesn't mutate shared state.

**Why:** In the OCR submit flow, the page needs to preserve the captured image during the paywall interaction and retry with a cache-bust header. `upgradeTo()` abstracts away too much and doesn't support retry semantics.

**How to apply:** When a feature-gated action fails and needs paywall → retry, handle paywall at the page level using `showPaywallCallbackProvider`, not `upgradeTo()`.

---

**Legacy endpoints can be compatibility bypasses.** `/extract` (sync OCR) doesn't create `ocr_scans` rows, so its quota check isn't tied to the authoritative async ledger. Document exempt behavior explicitly and monitor usage for sunset.

**Why:** The rate limiter queries `ocr_scans` for counts, but `/extract` never inserts rows. Its rate limiting is advisory at best.

**How to apply:** When adding hard-wall enforcement, audit ALL endpoints that touch the same resource. Document any that are exempt.
