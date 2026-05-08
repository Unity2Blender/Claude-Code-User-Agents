---
name: JSONB shape variants decode as Map, not strict typed model
description: When modeling a JSONB column whose canonical shape is locked elsewhere and may legitimately omit subfields, use Map<String, dynamic>? rather than reusing a strict typed shape. Strict models silently drop variant payloads.
type: feedback
originSessionId: 0adda812-a90b-43e0-ba00-4a0baa85afbb
---
When the Flutter `TutorialVideo` model gained `previewCtaConfig` (the JSONB sibling of `cta_config`), the user explicitly directed: model it as `Map<String, dynamic>?`, NOT as `CtaConfig`. The reasoning: `preview_cta_config` is locked reel-watch metadata that may not carry every field a strict `CtaConfig` decoder requires (e.g., `prefix`, `label_override`). A strict typed decode would either throw on the legitimate variant or — worse — silently drop the row at codegen layer.

**Rule:** When modeling a JSONB column whose authoring shape is locked by a different code path or content-ops contract, default to `Map<String, dynamic>?`. Apply the strict typed shape (`CtaConfig.fromCanonicalJson`, `CtaConfig.fromOverrideJson`) only at the consumer that needs the typed semantics, not at the carrier model.

**Why:** Two-layer model architecture from `flutter-architecture-principles`. The carrier model is a row schema; the consumer model is a domain semantic. Conflating them creates silent decode loss when the shape diverges across surfaces (canonical reel CTA vs. preview CTA vs. banner override).

**How to apply:** Before modeling a JSONB column as a strict typed class, ask: does every authoring surface for this column emit the SAME field set with the SAME contract? If yes, strict is fine. If any surface emits a subset (or a superset, or a renamed field), carry as `Map<String, dynamic>?` and convert at the consumer. This particularly applies to `cta_config` family columns, `target_params`, `illustration_overlay`, and any future tutorial JSONB sibling columns.

**Tolerance defaults:** When adding nullable Worker-emitted columns to a freezed model, prefer `@Default(false)` for booleans and explicit `String? / int? / Map?` for nullables. The Worker fallback playlist branch (`recommendations.ts:142-162`) maps `r.is_media_ready as boolean ?? null` — without `@Default(false)`, codegen would throw on null.
