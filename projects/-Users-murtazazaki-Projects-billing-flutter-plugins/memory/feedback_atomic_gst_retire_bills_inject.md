---
name: UI surface swaps ship direct unless user requests a flag
description: Supersedes the old DynamicFooter/BannerCard coexistence doctrine. Do not keep legacy and new UI paths behind flags unless explicitly requested.
type: feedback
status: superseded-rewrite
originSessionId: 519b0f7a-5681-477f-81a4-0ccaf3994e4a
---

**Rule:** For UI surface swaps, ship the replacement directly as a small reversible diff unless the user explicitly asks for a runtime flag or coexistence window.

**Why:** The previous Banner v5 guidance preferred flag-gated coexistence for a prominent GST surface. That narrow rollout concern later polluted unrelated UI planning and encouraged dead branches. The current operator directive is simpler: UI is client-side and reviewable by the developer, so do not preserve old UI with runtime flags by default.

**How to apply:**
- Replace the UI path directly when the new UI is ready.
- Keep the old UI path only if it still has a real consumer or the user explicitly asks for a temporary flag/coexistence period.
- Use tests, visual review, and git/hotfix rollback instead of remote-config UI branching.
- If metrics need comparison, instrument the new UI directly; do not keep two UI implementations solely for optionality.
- When the old UI path has zero consumers, delete it in the same PR per `feedback_no_consumer_delete_now.md`.

**Superseded history:** The original memory came from retiring `DynamicFooter` in favor of `BannerCard`. That project history remains useful when debugging Banner v5, but it is no longer active doctrine for future UI work.
