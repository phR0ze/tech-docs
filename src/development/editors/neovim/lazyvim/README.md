# LazyVim <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

Vim configuration is composed of rabbit holes within rabbit holes and you can go as far down as you 
have oxygen to the point were you might suffer from decompression sickness after such an encounter.

The folks over at [LazyVim](https://www.lazyvim.org/) are trying to make Vim more approachable for 
the uninitiated. They are building a distribution of Vim powered by their self named plugin manager 
[lazy.nvim](https://lazy.folke.io/) that will provide a clean set of defaults to get Vim up and 
running for the modern developer.

### Quick links
- [.. up dir](../README.md)
- [Overview](#overview)

## Overview
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

### NixOS approach
The Nix and LazyVim approaches are at odds with each other as LazyVim is designed to pull 
dependencies as needed at runtime while the Nix way would be to build a custom Neovim package 
that includes all plugins and configuration at build time.

* ***Neovim derivation*** to package Neovim and plugins
* ***Plugin provisioning*** have Nix fetch/build the plugins so that Lazy doesn't try clone at runtime.
* ***Runtime config*** can be created in `~/.config/nvim` and writable for Lazy's `lazy-lock.json`
* ***Lazy configuration*** can be done via Lua to call `require("lazy").setup(...)` with correct paths
* ***Disable runtime installers***
* ***Treesitter grammers*** need to be prebuilt

### Nix Helpers
* ***Lazy-Nix-Helper*** allows you to map plugin names to Nix store paths for Lua config paths
  * e.g. `dir = require("lazy-nix-helper").get_plugin_path("plugin-name")`
* ***nixPatch-nvim*** takes your existing `lazy.nvim` config and patches it at build time to replace 
  plugin Git URLs with local (store) paths
* ***Neovim‑flake*** projects that mimic LazyVim’s layout/configs and provide derivations for Neovim 
  with plugin set
