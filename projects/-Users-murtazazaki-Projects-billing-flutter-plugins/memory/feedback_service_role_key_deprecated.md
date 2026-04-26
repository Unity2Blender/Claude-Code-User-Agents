---
name: Service Role Key Deprecated
description: SUPABASE_SERVICE_ROLE_KEY is deprecated and risky — document warnings in all microservices and skill rules
type: feedback
---

Service role key (`SUPABASE_SERVICE_ROLE_KEY`) is DEPRECATED and should NEVER be used in new code.

**Why:** Service role bypasses ALL RLS policies — a single leaked key gives full database access. The user explicitly flagged this as risky and wants it documented as a deprecation/warning in:
1. `design-postgres-tables` skill CONNECTION rules
2. `bill-ocr` microservice CLAUDE.md
3. `payment-gateway` microservice CLAUDE.md (both root and packages/)

**How to apply:** When reviewing or writing code that creates Supabase clients:
- Flag any new usage of service role key as a concern
- Prefer user JWT-based auth or custom service tokens where possible
- Existing usage in payment-gateway Workers is a known debt, not a pattern to copy
- Document the risk prominently near any secret configuration
