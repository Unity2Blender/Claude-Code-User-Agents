---
name: Plan-mode sequencing must not pre-authorize commit/push; user authorizes per-milestone after green tests
description: ExitPlanMode allowedPrompts must NOT include git commit/push. User explicitly authorizes commit/push at named milestones, gated on flutter test passing.
type: feedback
originSessionId: e92ff6ad-9951-46b2-b2c0-38710165b402
---
When exiting plan mode, the `ExitPlanMode.allowedPrompts` list must NOT include `git add / git commit / git push` prompts. The user explicitly removes those from plan-mode sequencing.

**Why:** The user interrupted ExitPlanMode v1 because allowedPrompts included pre-authorized git commands. Their stance: "Plan-mode sequencing must not pre-authorize commit/push commands. 'Commit/push only after explicit implementation authorization.'" The reasoning: commit/push is an irreversible publishing action that they want to control per-milestone after seeing tests pass.

**How to apply:**

1. **In plan files**: Sequencing sections explicitly state NO pre-authorized commit/push. Add a locked decision row noting "Commits and pushes are NOT pre-authorized. After completing each logical milestone, wait for user direction on commit + push."

2. **In ExitPlanMode**: `allowedPrompts` should only include implementation tools (build_runner, flutter test, flutter analyze, file edits). Do not include git commit/push.

3. **Per-milestone workflow** (user-authorized for KPI Checkpoint plan, milestones at Step 0 / 14 / 23 / 32 / 38):
   - Implement step
   - Run `flutter test -t critical` (or appropriate scope) to verify no regression
   - Fix any test failures
   - Commit + push (user has pre-authorized for these specific milestones)
   - Move to next milestone

The user said: "I am authorizing commits and pushes after step 0, step 14, step 23, step 32, step 38. The reason why is that before each checkpoint... you have to perform the Flutter tests to ensure that it's stable. Whatever we have changed has not bleeded anything, nothing is red. And after fixing the tests or the changed code, you can have commit and pushes as well."

So the authorization gate is: **green tests → commit → push**. Tests-red → fix → recheck. Never commit+push with red tests.
