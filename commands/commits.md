---
name: commits
description: Show recent commit history with context summary
---

Display the last N commits from a git repository with an AI-generated context summary explaining "where we left off" in the development process.

## Command Arguments

When invoked, accept these parameters:
- **count** (default: 2): Number of recent commits to show
- **directory** (optional): Directory to check (default: current working directory or tagged directory)
- **verbose** (default: false): Show full commit details including files changed

## Data Sources

1. **Git Repository**: Detect `.git` directory in current/parent directories or tagged directory
2. **Git Log**: Execute `git log -n <count> --format=fuller` for detailed commit info
3. **Git Branch**: Execute `git rev-parse --abbrev-ref HEAD` for current branch

## Analysis Tasks

1. **Determine target directory**:
   - If directory is tagged in IDE → use tagged directory
   - Else if --directory argument provided → use that directory
   - Else → use current working directory

2. **Find git repository**:
   - Check if `.git` exists in target directory
   - If not found, walk up parent directories until `.git` is found or filesystem root is reached

3. **Execute git commands**:
   - Get current branch name
   - Get last N commits with full details
   - Parse commit info: hash, author, date, message (title + body)

4. **Generate context summary**:
   - Analyze commit messages to understand:
     - What feature/work was being developed
     - What was just completed
     - Current state of the work
   - Provide a "where we left off" summary in plain English

5. **Handle edge cases**:
   - Repository with 0 commits (just initialized)
   - Fewer commits than requested
   - Detached HEAD state
   - Not a git repository

## Output Format

Display in a clean, readable format:

```
Repository: /Users/user/project (branch: main)

📝 Last 2 Commits:
───────────────────────────────────────────────────────────────
a1b2c3d (2 days ago) - Author Name
feat: Add checkpoint-based incremental scraping

Implemented checkpoint system for 8-10x faster re-scraping.
Key features:
- Track latest review date per app
- Only fetch new reviews in subsequent scrapes
- Automatic checkpoint creation and updates

───────────────────────────────────────────────────────────────
e4f5g6h (3 days ago) - Author Name
docs: Update CLAUDE.md with year-wise organization

Added comprehensive documentation for the new data organization
structure. Includes examples and best practices.

───────────────────────────────────────────────────────────────

💡 Context: You were working on performance improvements to the
scraping system. The checkpoint feature was just added for much
faster incremental scraping, and documentation was updated to
reflect the new data organization structure. The focus appears to
be on making the system more efficient for ongoing monitoring.
```

If verbose mode is enabled, also show:
```
📁 Files changed: 5 files (+247, -32)
  - src/checkpoint_manager.py (new file)
  - src/scraper.py (+152, -12)
  - CLAUDE.md (+95, -20)
  - ...
```

## Implementation Guidance

```python
import subprocess
from pathlib import Path
from datetime import datetime

def find_git_root(start_dir):
    """Walk up directories to find .git"""
    current = Path(start_dir).resolve()
    while current != current.parent:
        if (current / '.git').exists():
            return current
        current = current.parent
    return None

def run_git_command(cmd, cwd):
    """Execute git command and return output"""
    result = subprocess.run(
        cmd,
        shell=True,
        capture_output=True,
        text=True,
        cwd=cwd
    )
    if result.returncode != 0:
        raise Exception(f"Git command failed: {result.stderr}")
    return result.stdout.strip()

# Determine target directory
# Priority: tagged_dir > args['directory'] > os.getcwd()
target_dir = tagged_directory if tagged_directory else (directory or os.getcwd())

# Find git repository
git_root = find_git_root(target_dir)
if not git_root:
    print(f"❌ Not a git repository: {target_dir}")
    print("   (searched up to filesystem root)")
    return

# Get current branch
try:
    branch = run_git_command('git rev-parse --abbrev-ref HEAD', git_root)
except:
    branch = "detached HEAD"

# Get commits with detailed format
# Format: hash|author|date|subject|body
git_format = "%h|%an|%ar|%s|%b"
cmd = f'git log -n {count} --format="{git_format}|||"'
log_output = run_git_command(cmd, git_root)

if not log_output:
    print(f"Repository: {git_root} (branch: {branch})")
    print("\n📝 No commits yet (empty repository)")
    return

# Parse commits
commits = []
for commit_block in log_output.split('|||'):
    if not commit_block.strip():
        continue
    parts = commit_block.strip().split('|', 4)
    if len(parts) >= 4:
        commits.append({
            'hash': parts[0],
            'author': parts[1],
            'date': parts[2],
            'subject': parts[3],
            'body': parts[4] if len(parts) > 4 else ''
        })

# Display commits
print(f"Repository: {git_root} (branch: {branch})\n")
print(f"📝 Last {len(commits)} Commit{'s' if len(commits) != 1 else ''}:")
print("─" * 63)

for commit in commits:
    print(f"{commit['hash']} ({commit['date']}) - {commit['author']}")
    print(commit['subject'])
    if commit['body'].strip():
        print(f"\n{commit['body'].strip()}\n")
    print("─" * 63)

# Generate AI context summary
# Analyze all commit subjects and bodies to understand the work
commit_messages = [f"{c['subject']}\n{c['body']}" for c in commits]
context = generate_context_summary(commit_messages)
print(f"\n💡 Context: {context}")

# Verbose mode: show file stats
if verbose:
    stats = run_git_command(f'git log -n {count} --stat --oneline', git_root)
    print(f"\n📁 Files Changed:\n{stats}")
```

## Context Summary Generation

Analyze the commit messages to generate an intelligent summary:

```python
def generate_context_summary(commit_messages):
    """
    Analyze commit messages and generate a plain-English summary
    of what was being worked on and where the work left off.
    """
    # Extract key themes:
    # - Type of work (feat/fix/docs/refactor)
    # - Subject matter (what feature/component)
    # - Progress indicators (completed/in progress)

    # Example logic:
    themes = []
    for msg in commit_messages:
        # Look for conventional commit types
        if msg.startswith('feat:'):
            themes.append(('feature', msg.split(':', 1)[1].strip()))
        elif msg.startswith('fix:'):
            themes.append(('bugfix', msg.split(':', 1)[1].strip()))
        elif msg.startswith('docs:'):
            themes.append(('documentation', msg.split(':', 1)[1].strip()))
        elif msg.startswith('refactor:'):
            themes.append(('refactor', msg.split(':', 1)[1].strip()))

    # Generate natural language summary
    if not themes:
        return "Recent commits show ongoing development work."

    # Build context based on themes
    summary_parts = []

    # Identify primary focus
    work_types = [t[0] for t in themes]
    if 'feature' in work_types:
        summary_parts.append("You were working on new features")
    if 'bugfix' in work_types:
        summary_parts.append("bug fixes were addressed")
    if 'documentation' in work_types:
        summary_parts.append("documentation was updated")

    return ". ".join(summary_parts) + "."
```

**Note**: The actual context summary should be generated by Claude itself by reading and understanding the commit messages, not by following rigid pattern matching. Claude should provide an insightful, human-readable explanation of what the recent work was about.

## Error Handling

- **Not a git repository**: Display friendly message with searched path
- **Git command fails**: Show error and suggest checking git installation
- **No commits**: Handle empty repository gracefully
- **Directory doesn't exist**: Fall back to current directory with warning
- **Detached HEAD**: Show commit hash instead of branch name
- **Permission errors**: Show clear error about access permissions

## Usage Examples

```bash
# Show last 2 commits (default)
/commits

# Show last 5 commits
/commits --count 5

# Check specific directory
/commits --directory ./my-feature-branch

# Verbose mode with file stats
/commits --count 3 --verbose

# Short form
/commits -n 10
```

## Edge Cases

### Empty Repository (No Commits)
```
Repository: /path/to/project (branch: main)

📝 No commits yet (empty repository)
```

### Only 1 Commit Available (requested 2)
```
Repository: /path/to/project (branch: main)

📝 Last 1 Commit:
───────────────────────────────────────────────────────────────
a1b2c3d (5 minutes ago) - Your Name
Initial commit

💡 Context: This is a brand new repository with just the initial commit.
```

### Not a Git Repository
```
❌ Not a git repository: /Users/user/some-folder
   (searched up to filesystem root)

Tip: Initialize a git repository with: git init
```

### Detached HEAD State
```
Repository: /path/to/project (detached HEAD at a1b2c3d)

📝 Last 2 Commits:
...

⚠️  Warning: You are in a detached HEAD state.
   Consider creating a branch: git checkout -b new-branch
```

## Implementation Notes

1. **Directory Resolution Order**:
   - Check if Claude has access to IDE-tagged directory (if applicable)
   - Use --directory argument if provided
   - Fall back to `os.getcwd()`

2. **Git Root Discovery**:
   - Start from target directory
   - Walk up parent directories until `.git` is found
   - Stop at filesystem root if not found

3. **Relative Time Display**:
   - Use git's built-in relative time format (`%ar`)
   - Examples: "2 days ago", "3 weeks ago", "5 minutes ago"

4. **Commit Message Parsing**:
   - Subject line (first line) should be bold or prominent
   - Body text should be indented or separated
   - Handle commits with no body gracefully

5. **Context Summary Intelligence**:
   - This is the key feature - don't just list commits
   - Read and understand the commit messages
   - Identify patterns (what component, what type of work)
   - Explain in natural language what was happening
   - Be insightful: "You were refactoring the auth system" not just "commits were made"

## Design Philosophy

**Simple but insightful**: The command should be quick to run and easy to read, but provide real value by helping developers understand where they left off in their work. The AI context summary is what makes this command useful beyond just running `git log`.
