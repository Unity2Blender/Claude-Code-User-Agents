---
name: UI runtime flags require explicit user request
description: Supersedes the old two-flag visual+IA rollback doctrine. Do not add runtime flags, kill switches, remote-config branches, or client toggles for UI/UX work unless the user explicitly asks.
type: feedback
status: superseded-rewrite
originSessionId: 6f570db4-b080-4d0e-ad2a-1cbeacc8c52b
---

**Rule:** Do not add runtime flags, kill switches, remote-config branches, or client toggles for UI/UX changes unless the user explicitly asks for one.

This applies to visual styling, layout, dashboard information architecture, navigation shell changes, banners, forms, wizard surfaces, paywall screens, cards, tickers, and other client-rendered UI surfaces. UI is client-side: ship the direct change, review it visually, test it where useful, and keep the diff small enough to revert or hotfix.

**Why:** 2026-04-25 memory-cleanse directive. The old two-flag rule caused Claude to preserve legacy UI paths, add unnecessary toggles, and inflate simple UI plans into flag matrices. The operator clarified: if a UI flag is needed, they will explicitly ask. Otherwise, do not make a big fuss out of client-side UI changes.

**How to apply:**
- If a plan proposes `*_enabled`, KV flags, remote config, kill switches, or two-path rendering for UI only, remove that machinery unless the user explicitly requested it.
- For UI changes, prefer small direct diffs, widget/golden tests when useful, manual visual review, and git/hotfix rollback.
- If UI work changes data source, routing, or count semantics, label it information architecture and add source/routing tests. That classification does NOT imply a runtime flag.
- If a non-UI backend, billing, entitlement, or data-integrity path needs a safety mechanism, scope that mechanism to the backend/data plane. Do not wrap the UI in a flag by default.
- Delete the old UI path once it has no consumer. Do not keep it for forensic optionality.

**Superseded history:** The previous version required separate runtime flags for combined visual+IA overhauls and a 4-combo flag matrix. That was a banner-era rollback lesson, not a general UI doctrine. It is superseded outside explicit user-requested flags.
