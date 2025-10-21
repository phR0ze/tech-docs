# Lz-n <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

Vim configuration is composed of rabbit holes within rabbit holes and you can go as far down as you 
have oxygen to the point were you might suffer from decompression sickness after such an encounter.

The folks over at [LazyVim](https://www.lazyvim.org/) are trying to make Vim more approachable for 
the uninitiated. They are building a distribution of Vim powered by their self named plugin manager 
[lazy.nvim](https://lazy.folke.io/) that will provide a clean set of defaults to get Vim up and 
running for the modern developer. As good as this project is, its built from a non NixOS perspective 
which poses a number of problems for a NixOS system.

Alternatively I've built out a custom configuration using Nix and [Lz.n](https://github.com/lumen-oss/lz.n)
that fits a similar need i.e. easy plugin management, lua configuration and lazy loading of plugins 
based on events.

### Quick links
- [.. up dir](../README.md)
- [Overview](#overview)

## Overview
Inspired by [Fernando Ayats's work](https://ayats.org/blog/neovim-wrapper) I've built out something 
similar providing plugin mangement via a standard standalone Nix derivation, lua configuration with 
myh own simple plugin and lazy plugin loading using [Lz.n](https://github.com/lumen-oss/lz.n) as 
recommended by Fernando.

**References**
* [Nvim bundle](https://github.com/jla2000/nvim-bundle)
* 



### Lazy Vim's features
`lazy.nvim` provides the following features:
* 📦 Manage all your Neovim plugins with a powerful UI
* 🚀 Fast startup times thanks to automatic caching and bytecode compilation of Lua modules
* 💾 Partial clones instead of shallow clones
* 🔌 Automatic lazy-loading of Lua modules and lazy-loading on events, commands, filetypes, and key mappings
* ⏳ Automatically install missing plugins before starting up Neovim, allowing you to start using it right away
* 💪 Async execution for improved performance
* 🛠️ No need to manually compile plugins
* 🧪 Correct sequencing of dependencies
* 📁 Configurable in multiple files
* 📚 Generates helptags of the headings in README.md files for plugins that don't have vimdocs
* 💻 Dev options and patterns for using local plugins
* 📊 Profiling tools to optimize performance
* 🔒 Lockfile lazy-lock.json to keep track of installed plugins
* 🔎 Automatically check for updates
* 📋 Commit, branch, tag, version, and full Semver support
* 📈 Statusline component to see the number of pending updates
* 🎨 Automatically lazy-loads colorschemes

### Vim events
| Event          | Trigger                                                                |
| -------------- | ---------------------------------------------------------------------- |
| `BufRead`      | When a buffer is read (e.g., opening a file)                           |
| `BufReadPre`   | Before a buffer is read                                                |
| `BufNewFile`   | When creating a new file                                               |
| `BufEnter`     | When entering a buffer                                                 |
| `BufWinEnter`  | When entering a window                                                 |
| `BufWritePre`  | Before writing a buffer                                                |
| `BufWritePost` | After writing a buffer                                                 |
| `InsertEnter`  | When entering insert mode                                              |
| `InsertLeave`  | When leaving insert mode                                               |
| `CmdlineEnter` | When entering the command-line                                         |
| `CmdlineLeave` | When leaving the command-line                                          |
| `VimEnter`     | When Neovim finishes loading                                           |
| `UIEnter`      | When the UI connects (used in GUIs/remote UIs)                         |
| `LspAttach`    | When an LSP client attaches to a buffer                                |
| `CursorHold`   | When the cursor stays still for a while                                |
| `CursorMoved`  | When the cursor moves                                                  |
| `ModeChanged`  | When the mode (normal/insert/visual/etc.) changes                      |
| `TermOpen`     | When opening a terminal buffer                                         |
| `FocusGained`  | When Neovim gains focus                                                |
| `FocusLost`    | When Neovim loses focus                                                |

```lua
event = { "BufReadPre", "BufNewFile" }
```
