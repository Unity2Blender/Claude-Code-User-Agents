---
name: Prefer direct R2 public URLs over Worker proxy for public CDN assets
description: When designing Worker endpoints, don't add a proxy hop for already-public R2 assets — R2 public URLs are edge-cached globally
type: feedback
originSessionId: ff28ab3e-695e-4a2b-9395-c64f699f9424
---
**Rule:** For public R2 assets (no auth needed), the Flutter client should fetch direct from `R2_PUBLIC_PREFIX/<key>` (or equivalent R2 public URL). Do NOT route through a CF Worker proxy endpoint unless there's a concrete reason (private bucket, signed URLs, per-user transforms, or need for additional cache layer beyond R2's built-in CDN).

**Why:** User's Round 10 decision on banner illustrations — they chose direct R2 over Worker proxy (`/v1/banners/illustration/:key`). Rationale: R2 public URLs are already cached at every Cloudflare edge PoP, so an intervening Worker only adds latency + bundle size + dead code for no cache benefit. Worker proxy layers are only worthwhile for private buckets, auth gating, or transforms.

**How to apply:**
- When planning a new Worker endpoint, ask: is this asset public?
- If yes → Flutter fetches direct; no Worker route for it.
- If private/signed → Worker proxies with auth + cache TTL override.
- Existing proxy endpoints for public assets should be treated as dead code and removed during cleanup.
- Banner v2 specifically removes `microservices/tutorials/src/routes/banner-illustration.ts` as part of the migration.

**Related memory:** `project_banner_system_v2_plan.md` (full decision context)
