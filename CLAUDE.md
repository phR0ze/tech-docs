# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A personal technical reference built with [mdBook](https://rust-lang.github.io/mdBook/). Content lives under `src/` as Markdown files. The table of contents is manually maintained in `src/SUMMARY.md`. The built static site goes to `book/`.

## Common Commands

```bash
# Build the book
mdbook build

# Serve locally with live reload (default: http://localhost:3000)
mdbook serve

# Check for broken links and other issues
mdbook test
```

## Structure

- `book.toml` — mdBook configuration (title, source dir, language)
- `src/SUMMARY.md` — master table of contents; every page must be listed here to appear in the built book
- `src/<topic>/README.md` — each section is a directory with a `README.md` as its landing page
- `book/` — build output, not committed

Top-level topic sections: `ai`, `cloud`, `data`, `databases`, `development`, `gaming`, `hardware`, `homelab`, `media`, `networking`, `productivity`, `security`, `system`, `virtualization`

## Adding Content

When adding a new page:
1. Create the file (e.g. `src/topic/subtopic/README.md`)
2. Add an entry to `src/SUMMARY.md` in the appropriate location — the indentation level determines nesting in the sidebar
3. Run `mdbook build` to verify it renders without errors

New top-level sections also need a parent entry in `src/SUMMARY.md` before the nested entries.

## Page Structure Convention

Every page follows this structure:

```markdown
# Title <img style="margin: 6px 13px 0px 0px" align="left" src="<relative-path-to>/data/images/logo_36x36.png" />

Brief description of the topic.

### Quick links
* [.. up dir](..)
* [Section One](#section-one)
  * [Subsection](#subsection)
* [Another Section](#another-section)

### Linked pages
* [Sub Topic](subtopic/README.md)

## Section One
...
```

**Logo path** — relative from the file to `src/data/images/logo_36x36.png`:
- `src/<topic>/README.md` → `../data/images/logo_36x36.png`
- `src/<topic>/<subtopic>/README.md` → `../../data/images/logo_36x36.png`
- `src/<topic>/<subtopic>/<page>/README.md` → `../../../data/images/logo_36x36.png`

**Quick links** rules:
- First entry is always `[.. up dir](..)` (links to parent directory)
- Anchor links use lowercase with hyphens: `[My Section](#my-section)`
- Subdirectory links (no `README.md` suffix) are valid for Quick links: `[Distros](distros)`
- Only link to anchors/pages that actually exist in the file or repo

**Linked pages** rules:
- Lists sub-pages with explicit `README.md` in the path: `[Sub Topic](subtopic/README.md)`
- Only include pages that exist as actual files

**Heading levels:**
- `#` — page title only (one per file)
- `##` — major sections
- `###` — subsections (also used for `Quick links` and `Linked pages` nav blocks)

**Formatting conventions:**
- `***term***` — bold-italic for key terms and strong emphasis
- `**label**` — bold for inline labels/headers (e.g. `**References**`, `**WARNING**`)
- `` `value` `` — inline code for commands, filenames, flags, and literal values
- Fenced code blocks with language hint (e.g. ` ```bash `) for multi-line commands
- External links go in a `**References**` block or inline

## Keeping Quick Links Current

After any edit to a page's headings or structure, update the `### Quick links` block to mirror the
current outline. The block must reflect the page as it exists after the edit — not before.

Rules:
- Every `##` heading gets a top-level entry
- Every `###` heading gets an indented entry under its parent `##`
- `[.. up dir](..)` stays as the first entry, always
- Anchor format follows the [GitHub Flavored Markdown spec](https://github.github.com/gfm/#links): lowercase, spaces replaced with hyphens, punctuation stripped
- Remove entries for headings that were deleted; add entries for headings that were added
- Do not list `### Quick links` or `### Linked pages` themselves as entries

## Link Validation

When editing any page, verify:
1. All `### Quick links` anchors match actual `##`/`###` headings in the same file (case-insensitive, spaces→hyphens)
2. All relative file links (`subtopic/README.md`, `../sibling/README.md`) point to files that exist under `src/`
3. The logo `src` path has the correct number of `../` segments for the file's depth under `src/`
4. Every page listed under `### Linked pages` exists as a file in the repo
