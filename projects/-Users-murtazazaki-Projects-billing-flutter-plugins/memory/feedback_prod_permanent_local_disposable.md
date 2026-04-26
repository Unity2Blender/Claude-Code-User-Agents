---
name: Prod is permanent, local is disposable — asymmetric risk appetite
description: Prod-vs-local risk asymmetry. Local Docker is meant to be reset, rebuilt, and thrown away. Prod is built to stay. Risk appetite for local-targeted changes must never bleed into prod-targeted changes.
type: feedback
originSessionId: 83fb8343-bb17-4517-a0c9-061d55c22211
---
**Rule:** Production and local Docker have asymmetric lifespans and asymmetric risk budgets. Plans that target local-Docker recovery must never spill DDL, data mutations, or ledger repairs into prod as a "while we're at it" convenience. When a plan touches both, separate the local-only steps from the prod-touching steps with a hard gate — and require the user's explicit go-ahead at that gate.

**Why:** During the v3 rebaseline (2026-04-26), the user articulated this directly after the squash commit landed: "the entire purpose of this plan was to link the local Docker with whatever is happening with the latest schema of the prod one ... It was never to hold back the prod or to cause any problems in prod for local environment. Prod versus local environment because local environments come and go, prods are here to stay." Local can be `db reset`'d at any time and rebuilt from scratch — its data has no users, no revenue, no compliance footprint. Prod schema/data has all of those. A reversible mistake on local is a coffee-break fix; the same mistake on prod is an incident, possibly with money or trust at stake. Treating them with the same risk appetite is a category error.

**How to apply:**
- For any plan that touches Supabase, default to: **local steps run freely, prod-touching steps require an explicit guarded gate with user typing the project ref or equivalent confirmation.**
- Never bundle "rebaseline local" + "repair linked ledger" + "push new migrations to linked" into one execution sequence. Split into phases with the user's go-ahead between phases.
- `supabase db reset`, `supabase migration up`, `supabase db diff`, dump-from-linked-into-local-file: all local-safe.
- `supabase migration repair --linked`, `supabase db push --linked`, anything writing to linked: **prod-touching, gate every time even if the user already approved it for an earlier step.**
- When the user pauses ("stop here", "commit + status report"), interpret it as preserving prod safety — do not interpret follow-up offers as authorization to proceed with prod-touching work.
- Audit the plan against this lens before execution: list every command, mark each `LOCAL` or `LINKED`. If anything `LINKED` runs without an explicit user gate immediately before it, the plan is wrong.
- This applies recursively to all subsystems with a prod/local split: pg_cron jobs, Cloudflare Workers (production vs preview), Razorpay (live vs test), Firebase Functions (deployed vs emulator), GCS/R2 buckets (prod vs dev). Same asymmetry, same gate discipline.
