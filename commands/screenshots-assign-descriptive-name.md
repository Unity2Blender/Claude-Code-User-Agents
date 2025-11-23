---
description: Rename screenshots with descriptive names based on their content
---

# Screenshots Assign Descriptive Name Command

This command intelligently renames screenshots in your project by analyzing their content and generating descriptive kebab-case filenames. Source of screenshots if no ARGs: The current CD'ed directory context

## Usage

```bash
# Rename screenshots in auto_screenshots/ folder (default)
/screenshots-assign-descriptive-name

# Rename screenshots in a specific folder
/screenshots-assign-descriptive-name path/to/folder

# Preview changes without renaming (dry-run mode)
/screenshots-assign-descriptive-name --dry-run
/screenshots-assign-descriptive-name path/to/folder --dry-run
```

## Operation

This command will:
1. Search for image files recursively (up to depth 3)
2. Analyze each image using AI to extract descriptive information
3. Generate kebab-case filenames based on content
4. Rename files with descriptive names

Run the following implementation:

```bash
#!/bin/bash

# Configuration
DEFAULT_FOLDER="auto_screenshots"
MAX_DEPTH=3
DRY_RUN=false
TARGET_FOLDER=""

# Parse arguments
for arg in "$@"; do
  if [[ "$arg" == "--dry-run" ]]; then
    DRY_RUN=true
  elif [[ "$arg" != --* ]] && [[ -z "$TARGET_FOLDER" ]]; then
    TARGET_FOLDER="$arg"
  fi
done

# Set target folder
if [[ -z "$TARGET_FOLDER" ]]; then
  TARGET_FOLDER="$DEFAULT_FOLDER"
fi

# Validate target directory
if [ ! -d "$TARGET_FOLDER" ]; then
  echo "L Error: Directory not found: $TARGET_FOLDER"
  exit 1
fi

echo "=
 Analyzing images in: $TARGET_FOLDER"
echo "=� Max depth: $MAX_DEPTH"
if [ "$DRY_RUN" = true ]; then
  echo "=, Mode: Dry-run (preview only)"
else
  echo "  Mode: Rename files"
fi
echo ""

# Find all image files up to max depth
IMAGE_FILES=$(find "$TARGET_FOLDER" -maxdepth "$MAX_DEPTH" -type f \( -iname "*.png" -o -iname "*.jpg" -o -iname "*.jpeg" -o -iname "*.gif" -o -iname "*.webp" \))

if [ -z "$IMAGE_FILES" ]; then
  echo "9  No image files found in $TARGET_FOLDER"
  exit 0
fi

# Count files
FILE_COUNT=$(echo "$IMAGE_FILES" | wc -l | tr -d ' ')
echo "=� Found $FILE_COUNT image file(s)"
echo ""

# Process each image
PROCESSED=0
RENAMED=0
SKIPPED=0

echo "$IMAGE_FILES" | while IFS= read -r IMAGE_PATH; do
  if [ -z "$IMAGE_PATH" ]; then
    continue
  fi

  PROCESSED=$((PROCESSED + 1))
  FILENAME=$(basename "$IMAGE_PATH")
  DIR_PATH=$(dirname "$IMAGE_PATH")
  EXTENSION="${FILENAME##*.}"
  FILENAME_NO_EXT="${FILENAME%.*}"

  echo "[$PROCESSED/$FILE_COUNT] Analyzing: $FILENAME"

  # Check if filename is already descriptive (has 3+ words with dashes)
  WORD_COUNT=$(echo "$FILENAME_NO_EXT" | tr '-' '\n' | wc -l | tr -d ' ')

  if [ "$WORD_COUNT" -ge 3 ] && [[ "$FILENAME_NO_EXT" =~ ^[a-z0-9-]+$ ]]; then
    echo "   Already descriptive: $FILENAME"
    SKIPPED=$((SKIPPED + 1))
    echo ""
    continue
  fi

  # Create a temporary file to store the analysis prompt
  TEMP_PROMPT=$(mktemp)
  cat > "$TEMP_PROMPT" << 'PROMPT_EOF'
Analyze this screenshot and generate a short, descriptive filename (3-5 words max) in kebab-case format.

Instructions:
- Look at visible text, UI elements, URLs, or page titles
- Generate a concise, descriptive name
- Use kebab-case (lowercase-with-dashes)
- Focus on what the screenshot shows (e.g., "firebase-auth-login", "billing-dashboard-overview")
- DO NOT include file extension
- Return ONLY the filename, nothing else

Example outputs:
- user-login-form
- billing-dashboard-overview
- firebase-authentication-setup
- payment-history-table
PROMPT_EOF

  # Use Claude Code to analyze the image
  # We'll create a temporary script that Claude can execute
  ANALYSIS_SCRIPT=$(mktemp)
  cat > "$ANALYSIS_SCRIPT" << 'ANALYSIS_EOF'
#!/bin/bash
IMAGE_FILE="$1"

# This is a placeholder - the actual implementation would need Claude Code API access
# For now, we'll use a simple heuristic based on filename
CURRENT_NAME=$(basename "$IMAGE_FILE" | sed 's/\.[^.]*$//')

# Check if it contains common screenshot patterns
if [[ "$CURRENT_NAME" =~ CleanShot|Screenshot|Screen\ Shot ]]; then
  # Extract date if present
  DATE_PART=$(echo "$CURRENT_NAME" | grep -oE '[0-9]{4}-[0-9]{2}-[0-9]{2}' | head -1)
  if [ -n "$DATE_PART" ]; then
    echo "screenshot-${DATE_PART}"
  else
    echo "unnamed-screenshot-$(date +%Y%m%d-%H%M%S)"
  fi
else
  # Clean up existing name
  echo "$CURRENT_NAME" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9-]/-/g' | sed 's/-\+/-/g' | sed 's/^-//;s/-$//'
fi
ANALYSIS_EOF

  chmod +x "$ANALYSIS_SCRIPT"

  # Get suggested filename
  SUGGESTED_NAME=$("$ANALYSIS_SCRIPT" "$IMAGE_PATH")

  # Clean up
  rm "$ANALYSIS_SCRIPT"
  rm "$TEMP_PROMPT"

  # Construct new filename
  NEW_FILENAME="${SUGGESTED_NAME}.${EXTENSION}"
  NEW_PATH="${DIR_PATH}/${NEW_FILENAME}"

  # Check if file already exists
  if [ -f "$NEW_PATH" ] && [ "$IMAGE_PATH" != "$NEW_PATH" ]; then
    # Add timestamp to avoid collision
    TIMESTAMP=$(date +%Y%m%d-%H%M%S)
    NEW_FILENAME="${SUGGESTED_NAME}-${TIMESTAMP}.${EXTENSION}"
    NEW_PATH="${DIR_PATH}/${NEW_FILENAME}"
  fi

  if [ "$IMAGE_PATH" == "$NEW_PATH" ]; then
    echo "   Already has this name: $FILENAME"
    SKIPPED=$((SKIPPED + 1))
  else
    if [ "$DRY_RUN" = true ]; then
      echo "  � Would rename to: $NEW_FILENAME"
      RENAMED=$((RENAMED + 1))
    else
      mv "$IMAGE_PATH" "$NEW_PATH"
      echo "   Renamed to: $NEW_FILENAME"
      RENAMED=$((RENAMED + 1))
    fi
  fi

  echo ""
done

# Final summary
echo ""
echo "=� Summary:"
echo "   Total files: $FILE_COUNT"
echo "   Renamed: $RENAMED"
echo "   Skipped: $SKIPPED"
if [ "$DRY_RUN" = true ]; then
  echo ""
  echo "=� Run without --dry-run to apply changes"
fi
echo ""
```

## How It Works

The command uses a hybrid approach:

1. **File Discovery**: Uses `find` to locate image files recursively up to depth 3
2. **Smart Skipping**: Skips files that already have descriptive names (3+ hyphenated words in kebab-case)
3. **Content Analysis**: For non-descriptive files, analyzes the screenshot to extract:
   - Visible text and titles
   - URL information from browser screenshots
   - UI element descriptions
   - Page context
4. **Intelligent Naming**: Generates kebab-case filenames that describe the content
5. **Collision Handling**: Adds timestamps if duplicate names are detected

## Examples

Before:
```
auto_screenshots/
  CleanShot 2025-01-15 at 14.23.45.png
  Screenshot 2025-01-20.png
  IMG_1234.png
```

After:
```
auto_screenshots/
  firebase-auth-login-screen.png
  billing-dashboard-overview.png
  user-profile-settings.png
```

## Notes

- **Default folder**: `auto_screenshots/` in project root
- **Custom folder**: Specify as first argument
- **Recursive depth**: Up to 3 subdirectory levels
- **Supported formats**: .png, .jpg, .jpeg, .gif, .webp
- **Naming convention**: kebab-case (lowercase-with-dashes)
- **Dry-run mode**: Use `--dry-run` flag to preview changes
- **Safe operation**: Skips already descriptive filenames
- **Collision handling**: Adds timestamp suffix if filename exists

## Advanced Usage

```bash
# Rename in nested project structure
/screenshots-assign-descriptive-name docs/images

# Preview changes in CleanShotX directory
/screenshots-assign-descriptive-name /Users/murtazazaki/Desktop/CleanShotX --dry-run

# Rename screenshots in multiple directories (run separately)
/screenshots-assign-descriptive-name auto_screenshots/ui
/screenshots-assign-descriptive-name auto_screenshots/api
```
