---
name: Ultraplan refinement before ExitPlanMode
description: User refines the plan file via an "Ultraplan" pass before approving — do not call ExitPlanMode immediately after writing the plan file
type: feedback
originSessionId: 883c3ae8-87c8-4136-be66-72eb9a7911ad
---
After writing the plan file in plan mode, do NOT call ExitPlanMode until the user confirms. The user runs a separate "Ultraplan" refinement pass on the plan file first.

**Why:** 2026-04-13 session — on Phase A3 banner planning, ExitPlanMode was rejected with "Plan being refined via Ultraplan — please wait for the result." Plan files get a second-pass review before the implementation gate.

**How to apply:** 
- When you finish writing the plan file in plan mode, announce the file is written and describe what's in it — do NOT immediately call ExitPlanMode.
- Wait for the user to either (a) request changes, (b) paste back a refined plan they want you to adopt, or (c) explicitly tell you to call ExitPlanMode.
- If the user sends a refined plan, read it carefully and overwrite the plan file with the refined content (plus any delta commentary they provide), then again wait for the next signal.
- Only call ExitPlanMode when the user explicitly approves or says "exit plan mode" / "ship it" / equivalent.
