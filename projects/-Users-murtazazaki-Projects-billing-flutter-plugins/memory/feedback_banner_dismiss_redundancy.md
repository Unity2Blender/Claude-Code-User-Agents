---
name: Banner dismiss CTA redundancy
description: When X close button exists on banners, never add a Dismiss text CTA — cognitive overload (Hick's Law 20.7% decision time increase)
type: feedback
originSessionId: b352baea-be19-4b54-a872-17f086754485
---
Banner dismiss CTA redundancy: when X close button exists at the banner's top-right, never add a "Dismiss" text CTA as a banner action — they do the same thing and cognitive-overload the user.

**Why:** User explicitly identified this in the NOTICE (missing cost) banner. X and "Dismiss" both set `_noticeBannerClosed = true`. Hick's Law: 3 choices → 2 choices = 20.7% faster decision. M3 banner spec also says "If a close affordance is present, a text dismiss button is redundant."

**How to apply:** When designing any banner with a close X button, the only CTAs should be action buttons (Set cost, Fix cost, Yes intentional). The X button is the sole dismiss mechanism. Rule codified as BANNER-004 in the ui-styling skill.
