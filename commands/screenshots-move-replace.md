---
description: Sync screenshots from CleanShotX directory to this directory, replacing existing ones
---

# Screenshots Move & Replace Command

This command syncs screenshots from your CleanShotX directory to the current directory's `auto_screenshots/` folder.

## Usage

```bash
# Sync only root-level screenshots (default)
/screenshots-move-replace

# Sync all screenshots including subdirectories
/screenshots-move-replace --recursive
/screenshots-move-replace -r
```

## Operation

Run the following bash script to sync screenshots:

```bash
#!/bin/bash

# Configuration
SOURCE_DIR="/Users/murtazazaki/Desktop/CleanShotX"
TARGET_DIR="auto_screenshots"
RECURSIVE_MODE=false

# Parse arguments
for arg in "$@"; do
  if [[ "$arg" == "--recursive" ]] || [[ "$arg" == "-r" ]]; then
    RECURSIVE_MODE=true
  fi
done

# Validate source directory
if [ ! -d "$SOURCE_DIR" ]; then
  echo "L Error: Source directory not found: $SOURCE_DIR"
  exit 1
fi

# Remove existing target directory
if [ -d "$TARGET_DIR" ]; then
  echo "=�  Removing existing $TARGET_DIR directory..."
  rm -rf "$TARGET_DIR"
fi

# Create fresh target directory
echo "=� Creating fresh $TARGET_DIR directory..."
mkdir -p "$TARGET_DIR"

# Copy images based on mode
if [ "$RECURSIVE_MODE" = true ]; then
  echo "=� Syncing all screenshots recursively..."

  # Find and copy all image files preserving directory structure
  cd "$SOURCE_DIR"
  find . -type f \( -iname "*.png" -o -iname "*.jpg" -o -iname "*.jpeg" -o -iname "*.gif" -o -iname "*.webp" \) -exec sh -c '
    for file; do
      target_path="'"$OLDPWD/$TARGET_DIR"'/$(dirname "$file")"
      mkdir -p "$target_path"
      cp "$file" "$target_path/"
    done
  ' sh {} +
  cd "$OLDPWD"

  FILE_COUNT=$(find "$TARGET_DIR" -type f | wc -l | tr -d ' ')
else
  echo "=� Syncing root-level screenshots only..."

  # Copy only root-level image files
  find "$SOURCE_DIR" -maxdepth 1 -type f \( -iname "*.png" -o -iname "*.jpg" -o -iname "*.jpeg" -o -iname "*.gif" -o -iname "*.webp" \) -exec cp {} "$TARGET_DIR/" \;

  FILE_COUNT=$(find "$TARGET_DIR" -maxdepth 1 -type f | wc -l | tr -d ' ')
fi

# Report results
if [ "$FILE_COUNT" -eq 0 ]; then
  echo "�  No image files found to copy"
else
  echo " Successfully synced $FILE_COUNT image file(s) to $TARGET_DIR/"

  # Show summary
  echo ""
  echo "=� Summary:"
  if [ "$RECURSIVE_MODE" = true ]; then
    echo "   Mode: Recursive (including subdirectories)"
  else
    echo "   Mode: Root-level only"
  fi
  echo "   Source: $SOURCE_DIR"
  echo "   Target: $(pwd)/$TARGET_DIR"
  echo "   Files copied: $FILE_COUNT"
fi
```

## Notes

- **Source**: `/Users/murtazazaki/Desktop/CleanShotX`
- **Target**: `auto_screenshots/` (in project root)
- **Operation**: Deletes existing target directory before copying (full replace)
- **File types**: .png, .jpg, .jpeg, .gif, .webp
- **Default mode**: Copies only root-level images
- **Recursive mode**: Copies all images preserving folder structure
