---
name: Push after every commit — do not batch
description: User wants git push to happen immediately after every commit, not held for end of session. Batched commits-without-pushes caused a force-push-required divergence on 2026-04-22 (main vs origin/main forked when the session's mid-push landed on origin while subsequent commit --amend rewrote locally).
type: feedback
originSessionId: 149be99b-9154-46c2-8f4c-87bb5791f778
---
Push every commit immediately after creating it — never batch commits for end-of-session push.

**Why:** On 2026-04-22 the session committed PR-A + PR-B locally without pushing (sandbox denied `git push origin main`). Between then and the next sandbox-allowed push, `7698f7d1` (the original pre-amend PR-B) landed on origin via a non-Claude path (IDE push / auto-push hook). A later `git commit --amend` rewrote local history to `306ed725`. `origin/main` diverged with `7698f7d1` (broken 000350 — missing sort_order/display_order/FK/GENERATED-trigger fixes) while local had the fixed chain. Force-with-lease push was the only resolution. Also — if the broken `7698f7d1` was actually run against prod before divergence discovered, the deploy would have failed identically to the original 000342 failure. Never leave a commit unpushed.

**How to apply:**
- After every `git commit` where the change is mergeable (tests pass, lint clean), immediately run `git push`.
- If sandbox denies the push, DO NOT silently continue. Escalate immediately via AskUserQuestion with options: (a) add Bash permission rule, (b) push to a feature branch, (c) user pushes manually. Record the denial in session notes.
- NEVER rely on "I'll push at session end" or "user will push." Divergence compounds silently.
- If `git commit --amend` is needed, it must happen BEFORE any push of the original commit — never amend after push, because that guarantees divergence.
- After amending, if the pre-amend SHA somehow reached origin (via IDE / hook / collaborator), force-with-lease is the correct resolution — but AskUserQuestion first per project CLAUDE.md "NEVER force push to main/master" rule. `--force-with-lease` (not `--force`) provides a safety net against clobbering an unseen push.
- For feature branches (not main), push freely — the safety rail only applies to main/master.
- Project CLAUDE.md global rule already states: "Once a plan has been implemented or Features implemented, Git commit & push with a brief message of changes (Like a Hook on Stop or SessionEnd)." This memory sharpens it from "session-end" to "per-commit."
