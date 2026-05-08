---
name: Use showWatermarkProvider / hasFeature(removeBranding) for entitlement bypass — NOT isPaid
description: subscriptionProvider.isPaid is !isFree without expiry check. For paywall bypass decisions use hasFeature(Feature.removeBranding) (expiry-aware) or the showWatermarkProvider helper.
type: feedback
originSessionId: e92ff6ad-9951-46b2-b2c0-38710165b402
---
`SubscriptionState.isPaid == !isFree` (per `lib/state/providers/subscription/subscription_provider.dart`) does NOT check `expiresAt`. Using it for entitlement bypass means an EXPIRED Silver/Gold user still bypasses paywalls — incorrect.

`SubscriptionState.hasFeature(Feature feature)` at line 196 DOES check expiry. The helper provider `showWatermarkProvider` at `subscription_provider.g.dart:611` (defined at line 527) returns `!state.hasFeature(Feature.removeBranding)` — this is the correct expiry-aware Silver+ entitlement check.

**Why:** During the KPI Checkpoint plan, I designed the bypass as `subscription.isPaid` because both Silver and Gold are paid. The user flagged this as P0: an expired Silver user would silently bypass KPI checkpoint paywall despite no longer being entitled.

**How to apply:**
- **Silver+ active bypass check**: `!ref.read(showWatermarkProvider)` OR `ref.read(subscriptionProvider).hasFeature(Feature.removeBranding)`
- **Gold-only checks** (e.g., firmLimit): use a dedicated check, NOT `isPaid` and NOT `hasFeature(Feature.removeBranding)`
- Free users: below threshold (will see paywall)
- Active Silver/Gold: bypass
- Expired Silver/Gold: see paywall (treated as free until renewal)

The doctrinal `lib/state/providers/subscription/CLAUDE.md` (NEW per KPI plan) documents this as the canonical pattern.
