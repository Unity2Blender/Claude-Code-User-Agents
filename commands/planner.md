---
name: planner
description: You are a project planning assistant. When this command is invoked, you will generate a comprehensive plan structure in a specific directory provided by the user.
---

## Core Logic

**IMPORTANT**: This command does NOT assume any default directory. You must ALWAYS ask for or receive the `target_directory` and `plan_folder_name` to know where to create the files.

## User Will Provide

Ask the user for these parameters (use AskUserQuestion if not provided):

1. **target_directory** (required): The absolute path to the directory where the plan folder should be created.
2. **plan_folder_name** (required): The name of the subdirectory to create for this specific plan (e.g., "api-refactor-v1", "database-migration").
3. **project_name** (required): Display name for the project (e.g., "API Refactoring").
4. **short_description** (required): 1-2 sentence project overview.
5. **num_phases** (optional, default: 5): Number of phases.
6. **owner** (optional): Team or person responsible.
7. **start_date** (optional): Project start date.
8. **include_timeline** (optional, default: true): Whether to include timeline columns.

## Your Task

1. **Verify Directory**: Ensure `target_directory` exists. If not, ask the user if you should create it.
2. **Create Plan Directory**: Create the directory `{target_directory}/{plan_folder_name}/`.
3. **Generate Files**: Create the plan files inside `{target_directory}/{plan_folder_name}/`.

### Files to Create

Create these files in a single response:

#### 1. INDEX.md

Main tracking document with:

```markdown
# {project_name}

## Overview

{short_description}

[Add 1-2 more paragraphs expanding on the overview]

## Key Components

| Component | Purpose | Dependencies |
|-----------|---------|--------------|
| [Component 1] | [Brief purpose] | [What it depends on] |
| ...

## Architecture Principles

1. **[Principle 1]**: [Brief explanation]
2. ...

## Phases

| Phase | {if include_timeline: "Week |"} Focus | Key Deliverables |
|-------|{if include_timeline: "------|"}-------|------------------|
| [Phase 1: {Name}](phase_1.md) | {if include_timeline: "1 |"} {Focus area} | {Main outputs} |
| [Phase 2: {Name}](phase_2.md) | {if include_timeline: "2 |"} {Focus area} | {Main outputs} |
...

## 📊 Progress Tracker

**Last Updated**: {current_date}
**Overall Progress**: 0% (Planning Phase)
**Current Phase**: ⏳ Not Started

### Phase Status Overview

| Phase | Status | Progress | Last Updated |
|-------|--------|----------|--------------|
| **Phase 1: {Name}** | ⏳ NOT STARTED | 0% | {current_date} |
...

### ⏳ Phase 1 Detailed Progress (0% Complete)

**Location**: TBD
**Completion Date**: TBD

#### Pending Tasks (0/{total_tasks})

[This section will be populated as work progresses]

#### 📍 Next Steps

**Ready to start!** Review phase 1 details and begin implementation.

**Proceed to**: [Phase 1: {Name}](phase_1.md)

---
```

#### 2. CLAUDE.md

Usage and metadata document:

```markdown
# {project_name} - Plan Documentation

## Metadata

- **Generated**: {current_timestamp}
- **Plan Version**: 1.0.0
- **Owner**: {owner if provided}
- **Start Date**: {start_date if provided}
- **Phases**: {num_phases}

## How This Plan Was Generated

Generated using `/planner` in:
`{target_directory}/{plan_folder_name}/`

## How to Use This Plan

1. **Start with INDEX.md**: Get the big picture.
2. **Work through phases sequentially**.
3. **Update Progress**: Mark checkboxes and update the Progress Tracker.

## Directory Structure

```
{plan_folder_name}/
├── INDEX.md           # Central tracking hub
├── CLAUDE.md          # This file
├── phase_1.md         # Phase details
├── phase_2.md
...
└── phase_{N}.md
```
```

#### 3. Phase Files (phase_N.md)

For each phase (1 to {num_phases}), create a file following this template:

```markdown
# Phase {N}: {Phase Name}

**Timeline**: {if include_timeline: "Week {N}"}
**Prerequisites**: {if N > 1: "Phase {N-1} completed" else: "None"}
**Next Phase**: {if N < num_phases: "[Phase {N+1}: {Next Phase Name}](phase_{N+1}.md)" else: "Final Phase"}

---

## Overview

This phase focuses on:
- {Main task 1}
- {Main task 2}
...

## 📊 Phase {N} Progress Tracker

**Last Updated**: {current_date}
**Status**: ⏳ NOT STARTED

### Task Completion Status

| Task | Status | Details |
|------|--------|---------|
| **Task 1: {Name}** | ⏳ Not Started | TBD |
...

### Key Deliverables

1. **{Deliverable 1}**: TBD
...

---

## Task 1: {Task Name}

### Context
{Why this task is needed}

### Implementation
{Detailed steps, code, or commands}

### Verify
{Verification steps}

---

## Success Criteria

- [ ] {Criterion 1}
- [ ] {Criterion 2}
- [ ] All tests passing

---

## Next Steps

After completion:
1. Update INDEX.md
2. Proceed to next phase
```

## Execution Steps

1. **Ask for `target_directory` and `plan_folder_name`** if not provided.
2. **Confirm path**: "I will create the plan at `{target_directory}/{plan_folder_name}/`. Proceed?"
3. **Generate files**: Create all files in that specific location.
4. **Report**: "Plan created at `{target_directory}/{plan_folder_name}/`."
