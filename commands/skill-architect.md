---
description: Generate SKILL.md files by analyzing reference directories. Creates standardized skill definitions with resource indexes following navigation-planner format.
argument-hint: [skill-dest-dir] [references-dir]
---

# Skill Architect

Design a SKILL.md file at the specified destination by analyzing the provided references directory.

## Arguments (Positional)

| Arg | Variable | Description
|-----|----------|------------- 
| 1st | `$1` | Skill destination directory  
| 2nd | `$2` | References directory path 

**Invocation:** `/skill-architect .claude/skills/my-skill/ READMEs/references/my-refs/`

## Reference Format: navigation-planner

Use `.claude/skills/navigation-planner/SKILL.md` as the canonical format:

- **Header**: YAML frontmatter with `name` and `description`
- **Title**: `# <Skill Title>` with one-line overview
- **When to Invoke**: 5-7 bullet points of trigger scenarios
- **Resource Index**: Table mapping `resources/<file>` → `Purpose`
- **Resource Finder**: Detailed entry per resource with `**Use when:**` and `**Contents:**`

## Workflow

### Step 1: Analyze References Directory (`$2`)

List and read files in `$2`:
- Extract filename, extension, and type (.md, .dart, etc.)
- Read first 50-100 lines to understand purpose
- Identify key headings/topics

### Step 2: Generate Header (from `$1`)

Derive skill name from `$1` directory name:

```yaml
---
name: <derived-from-$1-directory-name>
description: <synthesized-from-$2-content-max-200-chars>
---
```

**Header Rules:**
- `name`: Lowercase with hyphens, derived from last folder in `$1` path
- `description`: Start with action verb ("Invoke when...", "Use for...")

### Step 3: Build Resource Index Table

| Resource | Purpose |
|----------|---------|
| `resources/<filename.md>` | <Brief purpose description> |
| `resources/<filename.dart>` | <Brief purpose description> |

### Step 4: Generate Resource Finder

For each resource file from `$2`:

```markdown
### <filename> - <Descriptive Title>
**Use when:** <Specific scenario when this resource is needed>

**Contents:** <Comma-separated key topics from file headings>
```

### Step 5: Handle Existing SKILL.md

- **If `$1/SKILL.md` exists**: Only update the `---` header block (preserve body content)
- **If new**: Generate complete SKILL.md following the schema

## Output

Write the complete SKILL.md content to: `$1/SKILL.md`

Ensure `$1/resources/` subdirectory exists or will contain the reference files from `$2`.

## Constraints

| Element | Limit |
|---------|-------|
| Header `description` | 200 chars max |
| Overview line | 150 chars max |
| When to Invoke items | 5-7 bullet points |
| Resource Index purpose | 80 chars per row |
| Resource Finder per entry | 200 chars combined |
| Total overview section | 2000 chars max |
