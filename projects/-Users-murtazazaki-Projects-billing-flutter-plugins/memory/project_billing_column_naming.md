---
name: Billing table column naming convention
description: billing.user_id vs users.firebase_uid — same Firebase UID, different column names. Production bug in tutorials Worker from copy-paste confusion.
type: project
originSessionId: 3e902d61-2d3b-4a8f-a30b-ec507acd73e4
---
The `billing` table stores Firebase UID as `user_id` (TEXT, UNIQUE), NOT `firebase_uid`. The `users` and `razorpay_subscriptions` tables use `firebase_uid` for the same value. This naming divergence is intentional (billing table predates the convention).

**Why:** 2026-04-09 production bug — tutorials Worker queried `billing.firebase_uid` (doesn't exist) instead of `billing.user_id`. PostgREST error 42703. Silent failure: function returned null on error (same as "user not found"), views route returned 200 OK, per-user view tracking data lost for all signed-in users.

**How to apply:** All TypeScript microservices now have `schema-constants.ts` with `BILLING.USER_ID`. Never hardcode column names as raw strings. pgTAP advisory guard test `hasnt_column('public', 'billing', 'firebase_uid')` exists. Dart side uses generated `Billing.c_userId`.
