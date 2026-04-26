---
name: Never create branches OR worktrees without explicit user permission
description: Default to committing on the loaded branch. Never `git checkout -b`. Never `EnterWorktree` unless the user explicitly says "worktree". If isolation seems needed, ASK first — do not infer authorization from "open a PR" or kernel rules about isolation.
type: feedback
originSessionId: 29573519-30ff-4263-9565-9f2fa2074e35
---
**Rule:** Never create a new git branch. Never call `EnterWorktree` unless the user explicitly says "worktree" or "use a worktree" in this session. If isolation appears needed, **ask first** via AskUserQuestion — do not infer authorization from the user saying "open a follow-up PR" or from kernel rules that mention worktrees as the preferred isolation mechanism.

**Why:**
- 2026-04-23 — `git checkout -b` for the 000362 hotfix forced cleanup (ff-merge back to main + delete local + delete remote). Created a memory to stop creating branches.
- 2026-04-26 — extended the same mistake into `EnterWorktree`. User said "open a follow-up PR" for the banner refactor; I jumped to `EnterWorktree` because the kernel rule says "for isolation use worktrees". User pushed back: "who had asked you to create a worktree without my permission?". The fix path: squash-merge the PR into main, exit + remove the worktree, delete the local branch.
- The kernel rule "for isolation use worktrees" describes the *preferred mechanism* if the user has already authorized isolation. It does NOT authorize me to decide isolation is needed. The default — even when the user explicitly mentions a PR — is commit on the loaded branch and let the user pick the workflow.

**How to apply:**
- **Default:** commit to the currently-loaded branch (usually `main`) and push there. This applies to features, hotfixes, refactors, migrations — everything.
- **Never (without explicit user authorization):** `git checkout -b`, `git switch -c`, `EnterWorktree`, or `Agent(isolation: "worktree")`.
- **"Open a PR" is not authorization for a worktree.** It's authorization to push to a branch and open a PR. Ask which branch — or just commit on main if that's where the session loaded.
- **If isolation truly seems necessary:** ask via AskUserQuestion. Phrasing: "Should I (a) commit on main and push direct, (b) push to a feature branch and open a PR, or (c) work in an isolated worktree?". Default option in the question: (a).
- **When pushing:** push to the branch the session is on. Never `-u origin new-name` to invent a remote branch.
- **If asked to ship across multiple branches:** ask the user to load each target via their own worktree; don't create one myself.
