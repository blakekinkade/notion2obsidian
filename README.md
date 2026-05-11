<div align="center">
  <img src="logo.svg" alt="notion2obsidian logo" width="120" height="120">

  # Notion to Obsidian Migration Tool

  A high-performance CLI tool to migrate Notion exports to Obsidian-compatible markdown format.

  [![License](https://img.shields.io/badge/license-MIT-8250E7?style=flat&labelColor=262626)](LICENSE)
  [![Tests](https://img.shields.io/badge/tests-122_passing-00B863?style=flat&labelColor=262626)](notion2obsidian.test.js)

</div>

## What It Does

Takes a Notion export (folder or zip) and converts it into a clean Obsidian vault:

1. **Strips Notion IDs** from all filenames and folder names (e.g. `My Note abc123def456.md` → `My Note.md`)
2. **Adds YAML frontmatter** with title, tags, aliases, notion-id, and any inline Notion properties
3. **Converts markdown links** to Obsidian wiki links (`[text](file.md)` → `[[file|text]]`)
4. **Consolidates attachments** — moves all images and files into a single `_attachments/` folder and updates all references
5. **Converts Notion callouts** to Obsidian callout format
6. **Processes CSV databases** — creates Dataview-compatible index pages
7. **Handles duplicates** — disambiguates files with the same name using folder paths in frontmatter

## Requirements

This tool uses Bun-specific APIs and must be run with [Bun](https://bun.sh), not Node.js.

```bash
curl -fsSL https://bun.sh/install | bash
```

## Setup

```bash
git clone https://github.com/blakekinkade/notion2obsidian.git
cd notion2obsidian
bun install
```

## Usage

```bash
bun run notion2obsidian.js <input> [options]
```

### Input

The input can be a folder or a zip file:

```bash
# Process a folder
bun run notion2obsidian.js /path/to/notion-export

# Process a zip file
bun run notion2obsidian.js ./Export-abc123.zip

# Process multiple zip files
bun run notion2obsidian.js *.zip
```

### Options

| Flag | Description |
|------|-------------|
| `-o, --output DIR` | Output directory (default: modifies input in place) |
| `-d, --dry-run` | Preview changes without modifying files |
| `-v, --verbose` | Show detailed processing info |
| `--no-callouts` | Disable Notion callout conversion |
| `--no-csv` | Disable CSV database processing |
| `--dataview` | Create individual MD files from CSV rows (default: index only) |
| `-h, --help` | Show help |

### Examples

```bash
# Preview what will happen (no changes made)
bun run notion2obsidian.js /path/to/export --dry-run

# Migrate a folder, output to a different location
bun run notion2obsidian.js /path/to/export -o ~/Obsidian/MyVault

# Migrate a zip file
bun run notion2obsidian.js ./Export-abc123.zip -o ~/Obsidian/MyVault

# Migrate multiple zips into one vault
bun run notion2obsidian.js *.zip -o ~/Obsidian/MyVault
```

> **Note:** The `-o` output flag works with zip files. For folder inputs, the tool currently modifies the folder in place — copy the folder first if you want to preserve the original.

## What Gets Transformed

### Filenames

```
Before: My Note abc123def456789012345678901234567.md
After:  My Note.md
```

### Frontmatter

Notion inline properties (key: value lines after the H1) are extracted and moved into YAML frontmatter:

```markdown
Before:
# My Note
Status: In Progress
Owner: Blake

Content here...

After:
---
title: My Note
tags: [folder-name]
aliases:
  - My Note abc123def456789012345678901234567
notion-id: abc123def456789012345678901234567
status: In Progress
owner: Blake
published: false
---

# My Note

Content here...
```

### Links

```markdown
Before: [See this doc](Other%20Note%20abc123.md)
After:  [[Other Note|See this doc]]

Before: [Section](Note%20abc123.md#heading)
After:  [[Note#heading|Section]]
```

### Attachments

Notion exports each page's attachments in a folder alongside the `.md` file:

```
Before:
  My Note abc123.md
  My Note/
    image.png
    document.pdf

After:
  My Note.md
  _attachments/
    my-note-image.png
    my-note-document.pdf
```

All references in `.md` files are updated to point to `_attachments/`.

### Callouts

```markdown
Before:
<aside>
<img src="https://www.notion.so/icons/token_blue.svg" width="40px" />
Important note here
</aside>

After:
> [!note] 📘 Important note here
```

### CSV Databases

When your Notion export includes CSV database files, the tool creates a Dataview-compatible index page:

```markdown
# Tasks

Database with 6 records.

**CSV File:** [[Tasks.csv|Open in spreadsheet app]]

## All Records

```dataview
TABLE WITHOUT ID Task name, Status, Due
FROM csv("Tasks.csv")
```
```

Use `--dataview` to also create individual MD notes for each CSV row.

## Migration Steps

The tool runs in two phases:

**Phase 1 — Analysis**
- Scans all `.md` files and directories
- Builds file map for link resolution
- Detects duplicate filenames
- Shows a preview of all planned changes

**Phase 2 — Execution**
1. Process file content: add frontmatter, extract inline properties, convert links, convert callouts
2. Skip md-into-folder moves (attachments handled in Step 5.5)
3. Rename directories (deepest first to avoid conflicts)
4. Rename individual files
5. Normalize image filenames
6. Consolidate all attachments into `_attachments/`, update references, delete empty folders
7. Process CSV databases

## Safety

- **Dry run mode** previews all changes without touching files
- **Confirmation prompt** requires pressing ENTER before any changes are made
- **No backups** — use `--dry-run` or copy your folder first
- **Duplicate detection** warns before migration and disambiguates using folder paths
- **Conflict resolution** appends a counter if a target filename already exists (e.g. `Note-1.md`)

## Testing

```bash
bun test
```

## Project Structure

```
notion2obsidian.js          # Main entry point
notion2obsidian.test.js     # Test suite (122 tests)
src/lib/
  utils.js                  # Shared utilities and regex patterns
  stats.js                  # Migration statistics
  cli.js                    # Argument parsing
  links.js                  # Link conversion
  callouts.js               # Callout transformation
  frontmatter.js            # Frontmatter generation and file processing
  scanner.js                # File traversal
  assets.js                 # User interaction and directory operations
  zip.js                    # Zip extraction
  csv.js                    # Database processing
  enrich.js                 # Notion API enrichment (experimental)
```

## License

MIT
