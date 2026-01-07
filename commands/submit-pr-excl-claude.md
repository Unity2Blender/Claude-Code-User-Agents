---
name: submit-pr-excl-claude
description: Submit PR to upstream excluding all Claude artifacts (completely hidden from reviewers)
allowed-tools: Bash(git:*), Bash(gh:*), Bash(rm:*), Bash(find:*), Bash(mkdir:*), Bash(cd:*), Bash(pwd:*), Bash(test:*), Bash(ls:*), Bash(cat:*), Bash(grep:*), Bash(xargs:*), Read, Grep, Glob
---

# Submit Clean PR to Upstream (Claude-Excluded)

Creating a pull request that **completely excludes** all Claude artifacts from reviewers.

## Context

- Current directory: !`pwd`
- Current branch: !`git branch --show-current`
- Working tree clean: !`test -z "$(git status --porcelain)" && echo "YES" || echo "NO - has uncommitted changes"`
- Upstream remote: !`git remote get-url upstream 2>/dev/null || echo "NOT_SET"`
- Upstream repo: !`gh repo view --json parent -q '.parent.owner.login + "/" + .parent.name' 2>/dev/null || echo "UNKNOWN"`
- Fork owner: !`gh repo view --json owner -q '.owner.login' 2>/dev/null || echo "UNKNOWN"`
- Commits ahead of upstream: !`git rev-list --count upstream/main..HEAD 2>/dev/null || git rev-list --count upstream/master..HEAD 2>/dev/null || echo "UNKNOWN"`
- Recent commits: !`git log --oneline -5 2>/dev/null || echo "NO_COMMITS"`

## Arguments

The command accepts these arguments:
- `$ARGUMENTS` - Feature name for PR branch (optional, auto-derives from current branch)

Parse arguments to determine:
- **feature-name**: First positional argument, or derive from current branch name
- **--squash**: Squash all commits into one (default behavior)
- **--preserve-commits**: Keep individual commit history
- **--no-worktree**: Use branch mode instead of worktree (fallback)

## Your Task

Submit a clean PR to the upstream repository with **ZERO trace of Claude usage**.

**Reviewers will NOT see:**
- `.claude/` folder
- Any `CLAUDE.md` files
- Modified `.gitignore`
- Any "deleted Claude files" commits
- Any mention of "claude" in commit messages

---

## Phase 1: Pre-flight Validation

**Run these checks (ALL must pass):**

```bash
# 1. Verify git repo
test -d .git && echo "OK: Git repository" || echo "FAIL: Not a git repo"

# 2. Verify upstream remote
git remote get-url upstream >/dev/null 2>&1 && echo "OK: Upstream configured" || echo "FAIL: No upstream"

# 3. Verify gh authenticated
gh auth status >/dev/null 2>&1 && echo "OK: GitHub authenticated" || echo "FAIL: Not authenticated"

# 4. Check working tree
test -z "$(git status --porcelain)" && echo "OK: Clean working tree" || echo "WARN: Uncommitted changes"

# 5. Check commits exist
UPSTREAM_BRANCH=$(git remote show upstream 2>/dev/null | grep "HEAD branch" | cut -d: -f2 | tr -d ' ')
COMMIT_COUNT=$(git rev-list --count "upstream/$UPSTREAM_BRANCH"..HEAD 2>/dev/null)
test "$COMMIT_COUNT" -gt 0 && echo "OK: $COMMIT_COUNT commits to submit" || echo "FAIL: No new commits"
```

**If validation fails:**
- No upstream: "Run `/fork-repo-contributions-prep` first to set up the repository"
- Not authenticated: "Run `gh auth login` to authenticate"
- Uncommitted changes: "Please commit or stash your changes first"
- No commits: "No changes detected compared to upstream. Make some commits first."

---

## Phase 2: Gather Information

```bash
# Get upstream info
UPSTREAM_REPO=$(gh repo view --json parent -q '.parent.owner.login + "/" + .parent.name')
UPSTREAM_BRANCH=$(gh repo view "$UPSTREAM_REPO" --json defaultBranchRef -q '.defaultBranchRef.name')
FORK_OWNER=$(gh repo view --json owner -q '.owner.login')
WORKING_BRANCH=$(git branch --show-current)

# Determine feature name
if [ -n "$1" ] && [[ ! "$1" =~ ^-- ]]; then
  FEATURE_NAME="$1"
elif [ "$WORKING_BRANCH" != "main" ] && [ "$WORKING_BRANCH" != "master" ]; then
  FEATURE_NAME=$(echo "$WORKING_BRANCH" | sed 's/[^a-zA-Z0-9-]/-/g')
else
  FEATURE_NAME=$(git log -1 --format=%s | cut -c1-40 | sed 's/[^a-zA-Z0-9]/-/g' | sed 's/--*/-/g')
fi

PR_BRANCH="pr/$FEATURE_NAME"
```

**Report:**
- Upstream: `{UPSTREAM_REPO}` (branch: `{UPSTREAM_BRANCH}`)
- Fork owner: `{FORK_OWNER}`
- Working branch: `{WORKING_BRANCH}`
- PR branch name: `{PR_BRANCH}`

---

## Phase 3: Fetch Latest Upstream

```bash
git fetch upstream "$UPSTREAM_BRANCH"
echo "Fetched latest from upstream/$UPSTREAM_BRANCH"
```

---

## Phase 4: Create Clean PR Environment

### Option A: Worktree Mode (Default - Recommended)

Creates an isolated directory for PR preparation, leaving your working directory untouched.

```bash
# Create worktree directory
WORKTREE_DIR=".pr-worktrees/$FEATURE_NAME"
mkdir -p .pr-worktrees

# Remove existing worktree if present
git worktree remove "$WORKTREE_DIR" 2>/dev/null || true
git branch -D "$PR_BRANCH" 2>/dev/null || true

# Create fresh worktree from upstream
git worktree add "$WORKTREE_DIR" -b "$PR_BRANCH" "upstream/$UPSTREAM_BRANCH"

echo "Created worktree at: $WORKTREE_DIR"
```

### Option B: Branch Mode (Fallback - use with --no-worktree)

Traditional approach that modifies your current directory.

```bash
# Store current branch
ORIGINAL_BRANCH=$(git branch --show-current)

# Stash any changes
git stash push -m "submit-pr-excl-claude: temporary stash" 2>/dev/null || true

# Delete existing PR branch if exists
git branch -D "$PR_BRANCH" 2>/dev/null || true

# Create fresh PR branch from upstream
git checkout -b "$PR_BRANCH" "upstream/$UPSTREAM_BRANCH"
```

---

## Phase 5: Copy Changes (Excluding Claude Files)

**CRITICAL**: Only copy files that are NOT Claude-related.

### For Worktree Mode:

```bash
cd "$WORKTREE_DIR"

# Get list of all changed files, excluding Claude artifacts
git diff --name-only "upstream/$UPSTREAM_BRANCH".."$WORKING_BRANCH" 2>/dev/null | \
  grep -v "^\.claude" | \
  grep -v "CLAUDE\.md$" | \
  grep -v "\.claude\.md$" | \
  while read -r file; do
    if [ -n "$file" ]; then
      # Ensure parent directory exists
      mkdir -p "$(dirname "$file")" 2>/dev/null || true
      # Checkout file from working branch
      git checkout "$WORKING_BRANCH" -- "$file" 2>/dev/null || true
    fi
  done

cd -  # Return to original directory
```

### For Branch Mode:

```bash
# Checkout all files from working branch
git checkout "$WORKING_BRANCH" -- .

# Immediately remove Claude artifacts
rm -rf .claude/ 2>/dev/null || true
find . -name "CLAUDE.md" -type f -not -path "./.git/*" -delete 2>/dev/null || true
find . -name "*.claude.md" -type f -not -path "./.git/*" -delete 2>/dev/null || true
```

---

## Phase 6: Restore Original .gitignore

**CRITICAL**: If `.gitignore` was modified locally, restore the upstream version.

```bash
# For worktree mode, work in worktree directory
WORK_DIR="${WORKTREE_DIR:-.}"

cd "$WORK_DIR"

# Check if .gitignore differs from upstream
if ! git diff --quiet "upstream/$UPSTREAM_BRANCH" -- .gitignore 2>/dev/null; then
  echo "Restoring original .gitignore from upstream"
  git checkout "upstream/$UPSTREAM_BRANCH" -- .gitignore 2>/dev/null || true
else
  echo ".gitignore unchanged from upstream"
fi

cd -
```

---

## Phase 7: Verify No Claude Artifacts

**MANDATORY verification before committing:**

```bash
WORK_DIR="${WORKTREE_DIR:-.}"
cd "$WORK_DIR"

echo "=== Verifying clean PR (no Claude artifacts) ==="

# Check for .claude directory
if [ -d ".claude" ]; then
  echo "WARNING: .claude/ directory found - removing"
  rm -rf .claude/
fi

# Check for CLAUDE.md files
CLAUDE_FILES=$(find . -name "CLAUDE.md" -not -path "./.git/*" 2>/dev/null)
if [ -n "$CLAUDE_FILES" ]; then
  echo "WARNING: CLAUDE.md files found - removing:"
  echo "$CLAUDE_FILES"
  find . -name "CLAUDE.md" -not -path "./.git/*" -delete
fi

# Check for .claude.md files
CLAUDE_MD_FILES=$(find . -name "*.claude.md" -not -path "./.git/*" 2>/dev/null)
if [ -n "$CLAUDE_MD_FILES" ]; then
  echo "WARNING: *.claude.md files found - removing:"
  echo "$CLAUDE_MD_FILES"
  find . -name "*.claude.md" -not -path "./.git/*" -delete
fi

# Check .pr-worktrees (should not be included)
if [ -d ".pr-worktrees" ]; then
  rm -rf .pr-worktrees/
fi

# Final verification
REMAINING=$(find . \( -name "CLAUDE.md" -o -name ".claude" -o -name "*.claude.md" \) -not -path "./.git/*" 2>/dev/null)
if [ -z "$REMAINING" ]; then
  echo "VERIFIED: No Claude artifacts found"
else
  echo "ERROR: Claude artifacts still remain:"
  echo "$REMAINING"
  exit 1
fi

cd -
```

---

## Phase 8: Stage and Commit

Work in the PR directory (worktree or current):

```bash
WORK_DIR="${WORKTREE_DIR:-.}"
cd "$WORK_DIR"

# Stage all changes
git add -A

# Check if there are changes to commit
if git diff --cached --quiet; then
  echo "ERROR: No changes to commit after excluding Claude files"
  exit 1
fi

# Show what will be committed
echo "=== Files to be included in PR ==="
git diff --cached --name-only

cd -
```

### Commit Message

Generate a commit message based on the changes. If squashing (default), create a comprehensive message:

```bash
cd "$WORK_DIR"

# Get commit subjects from working branch for the message
COMMIT_MESSAGES=$(git log --format="- %s" "upstream/$UPSTREAM_BRANCH".."$WORKING_BRANCH" 2>/dev/null | head -20)

# Create commit (user should provide or approve message)
git commit -m "feat: $FEATURE_NAME

Changes included:
$COMMIT_MESSAGES"

cd -
```

**Ask user to review/edit the commit message before finalizing.**

---

## Phase 9: Push to Fork

```bash
WORK_DIR="${WORKTREE_DIR:-.}"
cd "$WORK_DIR"

# Push to origin (the fork)
git push -u origin "$PR_BRANCH" --force-with-lease

echo "Pushed to origin/$PR_BRANCH"

cd -
```

---

## Phase 10: Create Pull Request

```bash
WORK_DIR="${WORKTREE_DIR:-.}"
cd "$WORK_DIR"

# Get file change stats
FILES_CHANGED=$(git diff --stat "upstream/$UPSTREAM_BRANCH"..HEAD | tail -1)

# Create PR
gh pr create \
  --repo "$UPSTREAM_REPO" \
  --head "$FORK_OWNER:$PR_BRANCH" \
  --base "$UPSTREAM_BRANCH" \
  --title "[TITLE - ask user or derive from feature name]" \
  --body "## Summary

[DESCRIPTION - ask user or derive from commits]

## Changes

$FILES_CHANGED

## Testing

[HOW TO TEST - ask user]
"

cd -
```

**Ask user for:**
1. PR title (suggest based on feature name)
2. PR description (suggest based on commit messages)
3. Any additional context for reviewers

---

## Phase 11: Cleanup and Summary

### For Worktree Mode:

```bash
# Return to original directory (already there)
echo ""
echo "=== PR Submission Complete ==="
echo ""
echo "PR worktree preserved at: $WORKTREE_DIR"
echo "To update the PR: cd $WORKTREE_DIR && make changes && git push"
echo "To remove after merge: git worktree remove $WORKTREE_DIR && git branch -D $PR_BRANCH"
```

### For Branch Mode:

```bash
# Return to original branch
git checkout "$ORIGINAL_BRANCH"

# Restore stash
git stash pop 2>/dev/null || true

echo "Returned to branch: $ORIGINAL_BRANCH"
```

### Final Report:

```
=== PR Submitted Successfully ===

PR URL: {PR_URL}

Branch: pr/{FEATURE_NAME} -> {UPSTREAM_REPO}:{UPSTREAM_BRANCH}
Commits: {N} commits (squashed/preserved)

=== Verification Results ===
  .claude/ folder:     NOT PRESENT
  CLAUDE.md files:     NOT PRESENT
  .gitignore:          ORIGINAL (from upstream)

The PR contains ZERO trace of Claude usage.

=== Next Steps ===
1. Review the PR on GitHub: {PR_URL}
2. Address any CI failures or review comments
3. After merge, cleanup:
   git worktree remove .pr-worktrees/{FEATURE_NAME}
   git branch -D pr/{FEATURE_NAME}
```

---

## Error Handling

| Error | Response |
|-------|----------|
| No upstream remote | "Run `/fork-repo-contributions-prep` first" |
| Uncommitted changes | "Please commit or stash changes first" |
| No commits to submit | "No changes vs upstream. Make commits first." |
| PR branch exists | Auto-delete and recreate |
| Push fails | "Check permissions. Try: `git push -u origin {branch} --force`" |
| PR creation fails | Show gh error, suggest manual creation |
| Cherry-pick conflicts | "Resolve conflicts or use squash mode" |
| Worktree creation fails | Fall back to branch mode |

---

## Important Reminders

1. **NEVER include Claude artifacts** - double-check before every commit
2. **NEVER modify upstream's .gitignore** - restore original if changed
3. **Keep PR clean** - no mention of "claude" in commit messages
4. **Preserve working directory** - worktree mode keeps your work untouched
5. **Verify before push** - always run the artifact verification step
