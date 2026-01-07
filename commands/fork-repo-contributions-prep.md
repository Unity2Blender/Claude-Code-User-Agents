---
name: fork-repo-contributions-prep
description: Set up a forked repo for open-source contributions with Claude Code (hides Claude files from upstream)
allowed-tools: Bash(git:*), Bash(gh:*), Bash(npm:*), Bash(yarn:*), Bash(pnpm:*), Bash(pip:*), Bash(bundle:*), Bash(cargo:*), Bash(go:*), Bash(composer:*), Bash(mkdir:*), Bash(cat:*), Bash(test:*), Bash(ls:*), Bash(cp:*), Read, Write, Glob, Grep
---

# Fork Repository Contributions Setup

Setting up this forked repository for clean open-source contributions with Claude Code integration.

## Context

- Current directory: !`pwd`
- Is git repo: !`test -d .git && echo "YES" || echo "NO"`
- Git remote origin: !`git remote get-url origin 2>/dev/null || echo "NOT_SET"`
- Git remote upstream: !`git remote get-url upstream 2>/dev/null || echo "NOT_SET"`
- Parent repo (if fork): !`gh repo view --json parent -q '.parent.owner.login + "/" + .parent.name' 2>/dev/null || echo "NOT_A_FORK_OR_NOT_DETECTED"`
- Current branch: !`git branch --show-current 2>/dev/null || echo "NO_BRANCH"`
- GitHub CLI authenticated: !`gh auth status 2>&1 | head -1`

## Your Task

Set up this forked repository for open-source contributions. Follow these phases:

---

## Phase 1: Validate Environment

**Perform these checks:**

1. **Verify git repository**: Check if `.git/` directory exists
2. **Verify GitHub CLI**: Ensure `gh` is installed and authenticated
3. **Verify this is a fork**: Use `gh repo view --json parent` to confirm parent exists

**If validation fails:**
- Not a git repo: "This directory is not a git repository. Clone your fork first with: `git clone <your-fork-url>`"
- gh not installed: "GitHub CLI (gh) required. Install from https://cli.github.com/"
- Not authenticated: "Please authenticate with: `gh auth login`"
- Not a fork: "This repository doesn't appear to be a GitHub fork. This command is designed for contributing to upstream repositories."

**Only proceed if ALL validations pass.**

---

## Phase 2: Configure Upstream Remote

1. **Get parent repository info:**
   ```bash
   PARENT_REPO=$(gh repo view --json parent -q '.parent.owner.login + "/" + .parent.name')
   PARENT_BRANCH=$(gh repo view "$PARENT_REPO" --json defaultBranchRef -q '.defaultBranchRef.name')
   ```

2. **Check if upstream remote exists:**
   - If exists and correct: Skip
   - If exists but wrong URL: Update with `git remote set-url upstream`
   - If not exists: Add with `git remote add upstream`

3. **Fetch upstream:**
   ```bash
   git fetch upstream
   ```

**Report:** "Upstream configured: upstream -> {PARENT_REPO} (default branch: {PARENT_BRANCH})"

---

## Phase 3: Detect Project Type

Analyze the repository to determine project type. Check for these files in order:

| File | Project Type | Package Manager |
|------|--------------|-----------------|
| `package-lock.json` | Node.js | npm |
| `yarn.lock` | Node.js | yarn |
| `pnpm-lock.yaml` | Node.js | pnpm |
| `package.json` (no lockfile) | Node.js | npm (default) |
| `requirements.txt` | Python | pip |
| `Pipfile` | Python | pipenv |
| `pyproject.toml` | Python | poetry/pip |
| `setup.py` | Python | pip |
| `Gemfile` | Ruby | bundler |
| `Cargo.toml` | Rust | cargo |
| `go.mod` | Go | go mod |
| `composer.json` | PHP | composer |
| `pom.xml` | Java | maven |
| `build.gradle` | Java/Kotlin | gradle |

**Report:** "Detected project type: {TYPE} with {PACKAGE_MANAGER}"

---

## Phase 4: Install Dependencies (Optional)

Ask the user if they want to install dependencies, then run the appropriate command:

| Package Manager | Install Command |
|-----------------|-----------------|
| npm | `npm install` |
| yarn | `yarn install` |
| pnpm | `pnpm install` |
| pip | `pip install -r requirements.txt` |
| pipenv | `pipenv install --dev` |
| poetry | `poetry install` |
| bundler | `bundle install` |
| cargo | `cargo build` |
| go mod | `go mod download` |
| composer | `composer install` |

**Skip if:** User declines or dependencies already installed (check for node_modules, venv, vendor, etc.)

---

## Phase 5: Configure Local-Only Ignoring

**CRITICAL**: Use `.git/info/exclude` to ignore Claude files locally without modifying `.gitignore`.

This ensures Claude files are never accidentally committed or pushed.

**Add these patterns to `.git/info/exclude`:**
```bash
# First, check if patterns already exist
if ! grep -q "Claude Code" .git/info/exclude 2>/dev/null; then
  cat >> .git/info/exclude << 'EOF'

# Claude Code - Local only (never committed to upstream)
# Added by /fork-repo-contributions-prep
.claude/
**/CLAUDE.md
.pr-worktrees/
EOF
  echo "Added Claude patterns to .git/info/exclude"
else
  echo "Claude patterns already in .git/info/exclude"
fi
```

**Verify:** Confirm patterns were added by reading `.git/info/exclude`.

---

## Phase 6: Create Claude Directory Structure

Create the `.claude/` folder structure:

```bash
mkdir -p .claude/commands
mkdir -p .claude/settings
```

---

## Phase 7: Create CLAUDE.md Files

### 7a. Root CLAUDE.md

Create a `CLAUDE.md` at the project root with:

```markdown
# Project: {PROJECT_NAME}

## Overview
{Extract from README.md or package.json description}

## Quick Reference
- **Upstream**: {PARENT_REPO}
- **Default Branch**: {PARENT_BRANCH}
- **Project Type**: {DETECTED_TYPE}
- **Package Manager**: {PACKAGE_MANAGER}

## Development Commands
- Install: `{INSTALL_COMMAND}`
- Test: `{TEST_COMMAND}` (detect from package.json scripts, Makefile, etc.)
- Build: `{BUILD_COMMAND}`
- Lint: `{LINT_COMMAND}`

## Architecture
{List key directories and their purposes based on project structure}

## Contribution Workflow
1. Work on your feature in any branch
2. Claude files are automatically ignored (via .git/info/exclude)
3. When ready to submit PR: `/submit-pr-excl-claude [feature-name]`
4. PR will contain ZERO trace of Claude usage

## Key Files
{List important files for understanding the codebase}
```

### 7b. Module CLAUDE.md Files (Auto-detect)

Scan for significant directories that warrant their own CLAUDE.md:

**Detection criteria:**
- Directories matching: `src/`, `lib/`, `app/`, `packages/`, `components/`, `services/`, `api/`, `core/`, `modules/`, `cmd/`, `internal/`, `pkg/`
- Directories with 5+ source files
- Monorepo packages (directories with their own package.json, Cargo.toml, etc.)

**For each detected directory**, create a CLAUDE.md with:
- Directory purpose (infer from name and file types)
- Key files in that directory
- Related directories/dependencies

**Limit:** Create at most 5 module CLAUDE.md files to avoid clutter. Prioritize by importance.

---

## Phase 8: Summary Report

Display a formatted summary:

```
=== Fork Contribution Setup Complete ===

Repository:  {CURRENT_REPO}
Upstream:    {PARENT_REPO} (branch: {PARENT_BRANCH})
Project:     {PROJECT_TYPE} ({PACKAGE_MANAGER})

Configuration:
  [x] Upstream remote configured
  [x] .git/info/exclude updated (local-only ignoring)
  [x] .claude/ directory created
  [x] Dependencies installed (or skipped)

CLAUDE.md Files Created:
  - CLAUDE.md (root)
  - src/CLAUDE.md
  - {other modules...}

=== Important ===
Claude files will NEVER appear in your upstream PRs!

Next Steps:
1. Start working on your feature
2. Commit normally (Claude files are auto-ignored locally)
3. When ready to submit PR: /submit-pr-excl-claude [feature-name]
```

---

## Error Handling

| Error | Response |
|-------|----------|
| Not a git repo | Exit with clone instructions |
| gh not installed | Exit with install link |
| Not authenticated | Exit with `gh auth login` instructions |
| Not a fork | Exit explaining this is for forked repos |
| Upstream fetch fails | Warn but continue (network issue) |
| Dependency install fails | Warn and continue |

---

## Important Notes

- **DO NOT modify `.gitignore`** - use only `.git/info/exclude`
- **DO NOT commit any Claude files** - they should remain local only
- **Focus on setup, not modification** - this command prepares, doesn't change source code
