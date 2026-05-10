---
name: publish-pages
description: Upload files to the romlam11/pages GitHub repository and publish them to pages.rol4.com. Use when the user wants to upload, publish, or deploy files to their GitHub Pages site. Handles HTML, CSS, JS, images, and other web assets.
---

# Publish to GitHub Pages

Upload files to the `romlam11/pages` repository (main branch) so they are served at `pages.rol4.com`.

## Repository Details

- **Repo**: `romlam11/pages`
- **Branch**: `main` (GitHub Pages source)
- **Live URL**: `https://pages.rol4.com/<path>`

## Workflow

Make a todo list for all the tasks in this workflow and work on them one after another.

### 1. Identify Files to Upload

Determine which files the user wants to publish. They may:
- Reference a local file path (read it with the Read tool or file MCP tools)
- Paste content inline in the conversation
- Ask you to create a new file from scratch

For each file, determine:
- **Destination path** in the repo (e.g. `my-page.html`, `assets/style.css`)
- **File content** — read or generate it

### 2. Read Local File Content (if applicable)

If the user references a local file, read its content using the Read tool or
`mcp__167566c6-d40f-4c0d-913a-4830c9a1fa83__read_file_content` for files
outside the working directory.

For binary files (images, fonts, etc.) use the download/read MCP tool to get
base64-encoded content.

### 3. Check for Existing Files (optional)

To avoid accidental overwrites on important files, use `mcp__github__get_file_contents`
to check if the destination path already exists:

```
repo: romlam11/pages
owner: romlam11
path: <destination-path>
```

If a file exists and the user hasn't confirmed overwriting it, ask before proceeding.

### 4. Upload Files

Use `mcp__github__push_files` to push one or more files in a single commit:

```
owner: romlam11
repo: pages
branch: main
message: "Publish <filename(s)>"
files: [
  { path: "<destination-path>", content: "<file-content-as-text>" }
]
```

For a single file you may also use `mcp__github__create_or_update_file`:

```
owner: romlam11
repo: pages
branch: main
path: <destination-path>
content: <base64-encoded-content>
message: "Publish <filename>"
sha: <existing-sha-if-updating>
```

> **Note**: `create_or_update_file` requires content Base64-encoded. For text
> files, encode the UTF-8 string. `push_files` accepts plain text directly.
> Prefer `push_files` for text files; use `create_or_update_file` for binary.

### 5. Confirm Publication

After a successful push, report to the user:

- Which files were uploaded
- The live URL(s) where each file is accessible:
  `https://pages.rol4.com/<destination-path>`
- Note that GitHub Pages may take 30–60 seconds to propagate changes

## Edge Cases

- **Overwriting**: If `get_file_contents` returns an existing file, pass its
  `sha` to `create_or_update_file` to perform an update instead of a create.
- **Subdirectories**: GitHub Pages serves the full path. A file uploaded to
  `projects/demo/index.html` is live at
  `https://pages.rol4.com/projects/demo/index.html`.
- **Root index**: Uploading `index.html` replaces the site root — confirm with
  the user before doing so.
- **CNAME / config files**: Do not modify `CNAME` unless the user explicitly
  asks to change the custom domain.

## Wrap Up

End your turn with a short summary:

- Files uploaded (name + destination path)
- Live URL for each file
- Any warnings (e.g. overwrite, propagation delay)
