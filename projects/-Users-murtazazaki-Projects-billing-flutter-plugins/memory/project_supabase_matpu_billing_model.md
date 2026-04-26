---
name: Supabase MATPU Billing Model
description: How Supabase counts Third-Party MAU for Firebase Auth users — anon-key-only requests are free, any JWT with sub claim triggers billing
type: project
---

Supabase Third-Party MAU billing works differently from regular MAU.

**How MATPU is counted:** Distinct `sub` claims from validated JWTs during API requests (NOT via Auth Server logs like regular MAU). Each unique Firebase UID that presents a JWT = 1 Third-Party MAU per billing cycle.

**Anon-key-only requests (no JWT) do NOT count.** When only `apikey` header is sent (no `Authorization: Bearer`), the API gateway mints an internal short-lived JWT with `anon` Postgres role. No user identity attached. Zero MAU billing.

**Firebase anonymous users WITH JWTs DO count.** Even with missing `role` claim (defaults to `anon` Postgres role for DB queries), the JWT validation triggers Third-Party MAU counting because a valid third-party JWT with unique `sub` was presented.

**Pricing (as of 2026):**
- Free: 50,000 Third-Party MAU (hard cap, no overage)
- Pro ($25/mo): 100,000 Third-Party MAU, overage $0.00325/MAU
- Team ($599/mo): 100,000 Third-Party MAU, same overage rate
- MAU, Third-Party MAU, and SSO MAU are SEPARATE pools

**Why:** This confirms the MATPU guard is critical. Without it, every anonymous GST Calculator user (90% of user base) presenting a Firebase JWT would count as a billable Third-Party MAU. At scale: 130K users × $0.00325 overage = ~$97.50/month in unnecessary billing.

**How to apply:** The MATPU transport-level guard (ADR-001) prevents this by ensuring `currentJwt` stays null for anonymous/no-identity users. The Cloudflare Worker pattern (service_role key server-side) is the recommended approach for serving public reference data (HSN codes, tutorials) without MAU impact.

**Source:** Supabase official docs, pricing page, GitHub discussions #16965, #35933, #20356 (researched 2026-04-04)
