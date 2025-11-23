---
name: qna
description: Deep dive Q&A and codebase exploration to gain context and understanding
permission-mode: plan
---

Initiate a structured Q&A and exploration session. This command helps you understand the codebase, gather context, and document your findings in a dedicated workspace. It is designed for investigation, not project management.

## Command Arguments

- **directory** (required): The absolute or relative path where the investigation session will be stored.
- **session_name** (optional): Name for the specific investigation folder. Defaults to a timestamped name.
- **question** (optional): The initial question or topic to investigate.

## Directory Structure

The command will create a simple workspace for your investigation:

```text
[directory]/
  └── [session_name]/
      ├── index.md        # The "Whiteboard": Summary of questions, insights, and findings
      └── scratchpad.md   # (Optional) Raw notes, code snippets, and exploration logs
```

## Functionality

1.  **Setup**:
    -   **CRITICAL**: `directory` is mandatory.
    -   Create a subdirectory `[session_name]` inside `directory`.
    -   Initialize `index.md` to serve as the central point of context.

2.  **Exploration Loop**:
    -   If a `question` is provided, start by analyzing the codebase to answer it.
    -   Use `index.md` to record:
        -   **Questions**: What are we trying to figure out?
        -   **Findings**: What have we learned so far?
        -   **Context**: Links to relevant files (`[file](path)`), code snippets, and architectural insights.
    -   This is a **living document**. Update it as you explore.

3.  **Goal**:
    -   Bridge the gap between the developer and the codebase.
    -   Recover context after a break.
    -   Deeply understand complex logic before making changes.

## Implementation Guidance

```python
import os
from pathlib import Path
import datetime

def setup_investigation(base_dir, session_name=None):
    """
    Sets up the investigation workspace.
    """
    if not base_dir:
        raise ValueError("The 'directory' argument is required.")

    base_path = Path(base_dir).resolve()
    
    if not session_name:
        timestamp = datetime.datetime.now().strftime("%Y%m%d_%H%M%S")
        session_name = f"investigation_{timestamp}"
    
    session_path = base_path / session_name
    session_path.mkdir(parents=True, exist_ok=True)
    
    return session_path

def init_session_files(session_path, question):
    """
    Creates the initial context files.
    """
    index_file = session_path / "index.md"
    if not index_file.exists():
        with open(index_file, "w") as f:
            f.write(f"# Investigation: {session_path.name}\n\n")
            f.write(f"**Focus**: {question or 'General Codebase Exploration'}\n\n")
            f.write("## 🧠 Context & Findings\n\n")
            f.write("_Use this section to document what you learn._\n\n")
            f.write("## ❓ Open Questions\n\n")
            if question:
                f.write(f"- [ ] {question}\n")

# Main Flow
# 1. Get 'directory' (Required)
# 2. Create session folder
# 3. Create index.md
# 4. Start exploring based on 'question'
```

## Usage Examples

```bash
# Investigate a specific bug or feature
/qna --directory ./references/investigations --question "Why are billing reports failing?"

# General exploration
/qna --directory ./references/docs --session_name "auth_system_study"
```