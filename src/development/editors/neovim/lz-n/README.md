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

## NixOS approach
The Nix and LazyVim approaches are at odds with each other as LazyVim is designed to pull 
dependencies as needed at runtime while the Nix way would be to build a custom Neovim package 
that includes all plugins and configuration at build time.

### LazyVim plugins
* Coding
  * `mini.pairs`
    * Auto pairs Automatically inserts a matching closing character when you type an opening character like ", [, or (.
    * Always inserts regardless of context
    * `nvim-autopairs` provides the same functionality but is context aware
  * `ts-comments.nvim`
    * Improves comment syntax, lets Neovim handle multiple types of comments for a single language, 
      and relaxes rules for uncommenting.
  * `mini.ai`
    * Extends the a & i text objects, this adds the ability to select arguments, function calls, 
      text within quotes and brackets, and to repeat those selections to select an outer text object.
  * `lazydev.nvim`
    * Configures LuaLS to support auto-completion and type checking while editing your Neovim 
      configuration.
* Colorscheme
  * `tokynight.nvim`
  * `catppuccin`
* Editor
  * `grug-far.nvim`
    * search/replace in multiple files
  * `flash.nvim`
    * Flash enhances the built-in search functionality by showing labels at the end of each match, 
      letting you quickly jump to a specific location. 
  * `which-key.nvim`
    *  which-key helps you remember key bindings by showing a popup with the active keybindings of 
       the command you started typing.
  * `gitsigns.nvim`
    *  git signs highlights text that has changed since the list git commit, and also lets you 
       interactively stage & unstage hunks in a commit.
  * `trouble.nvim`
    *  better diagnostics list and others
  * `todo-comments.nvim`
    * Finds and lists all of the TODO, HACK, BUG, etc comment in your project and loads them into a 
      browsable list.
* Formatting
  * `conform.nvim`
    * LazyVim formatting
  * `mason.nvim`
    * ?
* Linting
  * `nvim-lint`
    * Asynchronously calls language-specific linter tools and reports their results via the 
      vim.diagnostic module.
* LSP 
  * `nvim-lspconfig`
    lspconfig
* TreeSitter
  * `nvim-treesitter`
    * Treesitter is a new parser generator tool that we can use in Neovim to power faster and more 
      accurate syntax highlighting.
  * `nvim-ts-autotag`
    *  Automatically add closing tags for HTML and JSX
* UI
  * `bufferline.nvim`
    * This is what powers LazyVim's fancy-looking tabs, which include filetype icons and close 
       buttons.
  * `lualine.nvim`
    * Displays a fancy status line with git status, LSP diagnostics, filetype information, and more.
  * `noice.nvim`
    * Replaces the traditional lower left hand corner
    * Highly experimental plugin that completely replaces the UI for messages, cmdline and the 
      popupmenu.
  * `mini.icons`
    * icons
  * `nui.nvim`
    *  ui components
  * `snacks.nvim`
    * ?
* Utils
  * `persistence.nvim`
    * Session management. This saves your session in the background, keeping track of open buffers, 
      window arrangement, and more. You can restore sessions when returning through the dashboard.
  * `plenary.nvim`
    *  library used by other plugins

### Feature set
- LSP
- Help autocomplete

### Start with lz.n
There is some complexity in rebuilding LazyVim's configuration in the ordering of dependencies and 
configuration. So i'm going to try and be very explicit and prefix my plugin choices with a four 
digit prefix e.g. `0000` for no dependencies and order loosly there after with nested dependencies.

Before any of that though you need a lazy loading plugin. I'm using `lz.n` for its pure lua simple 
lazy loading only approach to be used with Nix as the package manager.

***References***
- [LazyVim config](https://github.com/LazyVim/LazyVim/tree/main/lua/lazyvim)
- [Lazyvim Nix](https://github.com/jla2000/lazyvim-nix)
- [Nvim-bundle](https://github.com/jla2000/nvim-bundle)
- [Neovim Wrapper](https://ayats.org/blog/neovim-wrapper)

See [my solution](https://github.com/phR0ze/nixos-config/blob/main/options/apps/system/neovim/default.nix)
for details, but suffice it to say I found some odd namespacing issues when using the `init.lua` Nix 
community solution. Additionally I wasn't thrilled about having no lua syntax highlighting. So 
instead I put all my configuration in a custom plugin I called `init` and built into my Nix package 
to be run on boot.

I keep my `lua/init.lua` fairly light and only load options, keymaps, autocmds and my lazy loader 
`lz.n`,
```lua
require("config.options")               -- Load options from ../lua/config/options.lua
require("config.keymaps")               -- load keymaps from ../lua/config/keymaps.lua
require("config.autocmds")              -- load keymaps from ../lua/config/autocmds.lua

vim.cmd.packadd("lz.n")                 -- don't rely on nvim to load lz.n
vim.cmd.packadd("vim-deus")             -- don't rely on nvim to load lz.n
require("lz.n").load("plugins")         -- load plugins from ./lua/plugins

vim.cmd.colorscheme("deus")
```

### 0000-mini-icons
At its foundation you have your icons plugin choice. Your choices here consist of 
`mini.icons` and `nvim-web-devicons`. 

See explanation further below but the short of it is that `mini.icons` and completely replace 
`nvim-web-devicons`. The only reason for using the older icon set would be if you prefer the Nerd 
Font variants instead of `mini.icons`'s selection,

**Configure**
```lua
return {
  {
    "mini.icons",                                     -- Lua result/pack/opt module name
    event = "DeferredUIEnter",                        -- Equivalent of VeryLazy
    after = function()
      require("mini.icons").setup()                   -- Lua module path
      require("mini.icons").mock_nvim_web_devicons()  -- Setup compatibility layer for older plugins
      require('nvim-web-devicons')                    -- Load mini.pairs compatibility layer
    end,
  },
}
```

**nvim-web-webions**
- traditional goto before `mini.icons`
- depends on Nerd Fonts to work correctly
- plugins that depend on it include:
  - `nvim-tree`
  - `lualine`
  - `telescope`
  - `bufferline`
  - `which-key.nvim`

**mini.icons**
- A modern, minimal and pure lua replacement for `nvim-web-devicons`
- Offers similar mappings and lookups as `nvim-web-devicons`
- Lightweight, in that it has no external dependencies
- Can patch itself into plugins expecting `nvim-web-devicons` thus eliminating the need for both

### 0100-snacks.nvim
[snacks.nvim](https://github.com/folke/snacks.nvim) is the next plugin that most will consider given 
its multi-function use case. Essentially it is a collection of quality of life plugins.

**Dependencies**
- `mini.icons`

| Plugin          | Description 
| --------------- | ----------------------------------------------------------------------------------
| `animate`       | Efficient animations including over 45 easing functions (library) 	
| `bigfile`       | Deal with big files 	‼️
| `bufdelete`     | Delete buffers without disrupting window layout 	
| `dashboard`     | Beautiful declarative dashboards 	‼️
| `debug`         | Pretty inspect & backtraces for debugging 	
| `dim`           | Focus on the active scope by dimming the rest 	
| `explorer`      | A file explorer (picker in disguise) 	‼️
| `git`           | Git utilities 	
| `gitbrowse`     | Open the current file, branch, commit, or repo in a browser (e.g. GitHub, GitLab, Bitbucket) 	
| `image`         | Image viewer using Kitty Graphics Protocol, supported by kitty, wezterm and ghostty 	‼️
| `indent` 	      | Indent guides and scopes 	
| `input`         | Better vim.ui.input 	‼️
| `keymap` 	      | Better vim.keymap with support for filetypes and LSP clients 	
| `layout` 	      | Window layouts 	
| `lazygit`       | Open LazyGit in a float, auto-configure colorscheme and integration with Neovim 	
| `notifier` 	    | Pretty vim.notify 	‼️
| `notify` 	      | Utility functions to work with Neovim's vim.notify 	
| `picker` 	      | Picker for selecting items 	‼️
| `profiler` 	    | Neovim lua profiler 	
| `quickfile`     | When doing nvim somefile.txt, it will render the file as quickly as possible, before loading your plugins. 	‼️
| `rename` 	      | LSP-integrated file renaming with support for plugins like neo-tree.nvim and mini.files. 	
| `scope`         | Scope detection, text objects and jumping based on treesitter or indent 	‼️
| `scratch`       | Scratch buffers with a persistent file 	
| `scroll`        | Smooth scrolling 	‼️
| `statuscolumn`  | Pretty status column 	‼️
| `terminal` 	    | Create and toggle floating/split terminals 	
| `toggle`        | Toggle keymaps integrated with which-key icons / colors 	
| `util`          | Utility functions for Snacks (library) 	
| `win`           | Create and manage floating windows or splits 	
| `words`         | Auto-show LSP references and quickly navigate between them 	‼️
| `zen`           | Zen mode • distraction-free coding

### 0100-which-key.nvim
[which-key.nvim](https://github.com/folke/which-key.nvim) shows available keybindings in a popup as 
you type to help you remember.

* `classic` much like modern but takes whole bottom portion of screen with no window effect
* `modern` provides a nicely rounded popup window at the bottom
* `helix` provides a cramped vertical window on the far right

**Dependencies**
- `mini.icons`

**Configuration**
```lua
return {
  "which-key.nvim",                                 -- Lua result/pack/opt module name
  event = "DeferredUIEnter",                        -- Equivalent of VeryLazy
  after = function()                                -- Function to load after the event
    require("which-key").setup({                    -- First lazy load by plugin name not Nix package name
      preset = "modern",                            -- Window layout [ classic | modern | helix ]
      delay = 300,
    })
  end,
}
```

### 0100-lualine.nvim
[lualine.nvim](https://github.com/nvim-lualine/lualine.nvim) is a blazing fast pure lua statusline 
plugin.

**Dependencies**
- `mini.icons`

**Configuration**
```lua
return {
  "lualine.nvim",                                   -- Lua result/pack/opt module name
  event = "DeferredUIEnter",                        -- Equivalent of VeryLazy
  after = function()                                -- Function to load after the event
    require("lualine").setup()
  end,
}
```

### 0100-bufferline.nvim
[bufferline.nvim](https://github.com/akinsho/bufferline.nvim) is a snazy pure lua buffer line

**Dependencies**
- `mini.icons`

**Configuration**

`project.nvim` - Detects the root of your project and sets that as the CWD for FZF

https://github.com/KijitoraFinch/nanode.nvim
https://github.com/2KAbhishek/nerdy.nvim
https://github.com/MeanderingProgrammer/render-markdown.nvim
https://github.com/Wansmer/symbol-usage.nvim
https://github.com/christoomey/vim-tmux-navigator
https://github.com/dmtrKovalenko/fff.nvim
https://github.com/folke/lazydev.nvim
https://github.com/ibhagwan/fzf-lua
https://github.com/j-hui/fidget.nvim
https://github.com/kylechui/nvim-surround
https://github.com/lewis6991/gitsigns.nvim
https://github.com/mfussenegger/nvim-lint
https://github.com/neanias/everforest-nvim
https://github.com/neovim/nvim-lspconfig
https://github.com/saecki/crates.nvim
https://github.com/saecki/live-rename.nvim
https://github.com/sindrets/diffview.nvim
https://github.com/stevearc/conform.nvim
https://github.com/Saghen/blink.cmp
https://github.com/catppuccin/nvim
