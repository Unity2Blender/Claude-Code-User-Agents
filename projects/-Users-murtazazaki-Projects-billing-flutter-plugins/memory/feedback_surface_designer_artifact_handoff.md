---
name: Surface-designer requires .surface.md artifact handoff
description: When the surface-designer skill is invoked as a hard gate, the deliverable is a separate .surface.md handoff artifact in the canonical location — embedded JSON inside a planning document does not satisfy the contract.
type: feedback
originSessionId: 2c5fcb68-16a2-4df2-b8c9-85d16216119a
---
P1 DEFAULT — Surface-designer hard gates require a `*.surface.md` handoff artifact, not embedded JSON inside a planning document.

**Why:** 2026-05-02 plan rejection. My plan embedded the surface-designer JSON gate contract directly in section 11 of the plan file. The user pointed out that surface-designer's skill contract requires a proper standalone `.surface.md` artifact at the canonical handoff location. Embedded JSON in a plan does not pass the gate; downstream regen tooling looks for the file, not for an inline block.

**How to apply:**
- When invoking surface-designer as a hard gate (page, sheet, component, wizard step body, redesign), produce a separate file with the full markdown structure surface-designer expects (sections, fields, states, accessibility, motion, test viewport, plus any pressure-test annotations).
- The canonical handoff location is `docs/specs/<surface>.surface.md` (or wherever surface-designer's skill rules currently mandate — verify in the skill's docs at decision time).
- The plan file references the artifact path; it does not duplicate the JSON.
- Use embedded JSON only for non-gated surface communication (informal walkthroughs, brainstorm docs, audit summaries).
- This applies recursively to wizard-designer (`*.wizard.md`), pdf-template-design (`*.pdf-template.md`), paywall-designer (`*.paywall.md`).
