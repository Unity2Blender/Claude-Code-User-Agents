---
name: Merge commit, not squash, for multi-CP feature branches
description: User prefers merge commits over squash when feature branch has meaningful CP granularity worth preserving on main
type: feedback
originSessionId: db9f8ed4-7e01-48cd-aec9-9508e435e39a
---
Default to `gh pr merge --merge` (merge commit), not `--squash`, for feature branches with 10+ commits where per-CP history is meaningful.

**Why:** User chose merge commit over squash for banner-impl (26 CPs) on 2026-04-13, explicitly preferring forensic history preservation even at the cost of a noisier `main` log. Rationale: each CP was deliberately scoped as an atomic commit boundary per the plan, so losing that granularity would remove `git blame`/`git bisect` precision. `git log --first-parent main` still gives a clean timeline when cleanliness matters.

**How to apply:**
- Feature branches with explicit per-CP commits (banner-v5, column-drift, etc.) → `--merge`
- Single-topic branches with 1-5 WIP commits → `--squash` is fine (no granular history to preserve)
- When in doubt on this project, ask via AskUserQuestion — the default is NOT obvious, and different branches get different treatment.

**Related:** `.claude/plans/mossy-singing-hippo.md` CP-25.
