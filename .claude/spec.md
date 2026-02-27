# tech-docs Repository Specification

## Overview
**tech-docs** is an [mdBook](https://rust-lang.github.io/mdBook/)-based personal knowledge
repository authored by phR0ze. It serves as a collection of research notes, reference material, and
documentation across a wide range of technology topics. The rendered book is built from Markdown
source files under `src/`.

## Build System
- **Tool**: mdBook
- **Config**: `book.toml` (title: "tech-docs", language: "en", source dir: "src")
- **Source**: `src/` directory containing all Markdown content
- **Output**: `book/` directory (generated, not committed)
- **Build command**: `mdbook build`
- **Dev server**: `mdbook serve`

## Directory Structure
All content lives under `src/`. Top-level categories:

| Category         | Description                                      |
| ---------------- | ------------------------------------------------ |
| `ai/`            | AI tools, agents, context engineering, platforms  |
| `cloud/`         | Cloud platforms (GCP)                             |
| `databases/`     | Database systems (MySQL, Redis, EER)              |
| `development/`   | Languages, editors, CI/CD, version control, UI    |
| `gaming/`        | Gaming platforms and tools                        |
| `hardware/`      | Graphics, printers, storage                       |
| `homelab/`       | Self-hosted services (media, photos, utils)       |
| `media/`         | Audio, video, encoding, editing, broadcast        |
| `networking/`    | DNS, VPN, proxy, remoting, routing, browsers      |
| `productivity/`  | Office tools, workspace                           |
| `security/`      | IAM, OAuth, crypto, password managers             |
| `system/`        | OS (NixOS, Android, OSX), DEs, window managers    |
| `virtualization/`| Containers, VMs, orchestration, networking        |

Shared assets:
- `src/data/images/` — logo and other image assets

## SUMMARY.md
`src/SUMMARY.md` is the mdBook table of contents. **Every page must have an entry here** to appear
in the rendered book. Entries are nested markdown list items with relative paths:

```markdown
- [Category](category/README.md)
  - [Topic](category/topic/README.md)
```

## README Conventions
Each topic directory contains a `README.md` following this structure:

### Title with Logo
```markdown
# Title <img style="margin: 6px 13px 0px 0px" align="left" src="../data/images/logo_36x36.png" />
```
The `src` path is relative to the file's location, pointing to `src/data/images/logo_36x36.png`.

### Quick Links Section
Anchor-based navigation within the page:
```markdown
### Quick Links
- [.. up dir](..)
- [Section Name](#section-name)
```

### Linked Pages Section
Links to child topic README files:
```markdown
### Linked pages
- [Child Topic](child_topic/README.md)
```

### Content Sections
Standard markdown sections (`##` headings) containing explanatory text, code blocks, tables, and
external reference links.

### Footer
Originally a footer was being used, but i'm pulling away from that and would like all footers of this
nature removed.
```markdown
<!-- vim: ts=2:sw=2:sts=2 -->
```

## Adding New Content

1. **Create the directory**: `src/<category>/<topic>/` using lowercase with underscores
2. **Create `README.md`** following the conventions above
3. **Add entry to `src/SUMMARY.md`** at the correct nesting level
4. **Add images** (if any) to `src/data/images/`

## Naming Conventions
- Directories: lowercase with underscores (e.g., `cloud_build`, `github_actions`)
- Content files: `README.md` for the main page of each topic
- Non-README pages: lowercase with underscores (e.g., `docker_container_action.md`)

## Git Conventions
- Commit messages are short descriptions of research activity
- Examples: "Researching vim and zed for rust", "VPN research", "Latest research"
