# Fuzzy Finding <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

Fuzzy finding provides and easy way to search for files, buffers, or any other lists and has rapidly 
become the goto for navigating in Neovim. Since I've become interested in upping my Neovim game the 
scene has been dominated first by `telescope` then by `fzf-lua` and now the Snacks plugin aggregate 
provides its own `picker` that provides similar functionality.

### Quick links
- [.. up dir](../README.md)
- [Snacks picker](#snacks-picker)
  - [Change picker layouts](#change-picker-layouts)
- [fzf-lua](#fzf-lua)
  - [fzf-lua dependencies](#fzf-lua-dependencies)
  - [fzf-lua configuration](#fzf-lua-configuration)

## Snacks picker
The Snacks picker mimics fzf-lua closely but integrates more tightly with the LazyVim Neovim 
distribution. I'm switching over to Snack's picker for fuzzy finding based on:

* I'm already using snacks for other reasons
* Snacks fuzzy finding seems to be on par with fzf-lua
* Snacks offers a backdrop option to dim the background which is awesome
* Snacks offers more control over the layout customization of the pickers

**References**
* [Snacks Explorer](https://github.com/folke/snacks.nvim/blob/main/docs/explorer.md)

### Keys
* Explorer picker keys
  * `<Tab>` - select files to work with
  * `a` - add a new file or a folder if it ends in `/`
  * `d` - delete the current file
  * `c` - to copy selected files
  * `m` - to move selected files
  * `r` - rename the current file

### Change picker layouts
The way to modify the picker's appearance it through changing the picker's layout. The picker plugin 
supports custom layouts.

```lua
require("snacks").setup({
  picker = {
    layout = {
      preset = "dropdown",                        -- drop down file picker only without preview
    },
    layouts = {
      dropdown = {                                -- Custom picker based on built in vscode preset
        hidden = { "preview" },                   -- don't want the preview
        layout = {
          backdrop = true,                        -- dims the backround making picker pop out
          row = 1,                                -- draw the picker this many rows from the top of the screen
          width = 0.8,                            -- percentage of screen width to use for picker
          min_width = 80,                         -- minimum number of columns to make the width of the picker
          max_width = 120,                        -- maximum number of columns to make the width of the picker
          height = 0.6,                           -- percentage of the screen height to user for picker
          border = false,                         -- don't draw the border as it collides with the search box
          box = "vertical",
          { win = "input", height = 1, border = true, title = "{title} {live} {flags}", title_pos = "center" },
          { win = "list", border = "hpad" },
          { win = "preview", title = "{preview}", border = true },
        },
      },
    },
  },
})
```

## fzf-lua
fzf-lua is a good fuzzy finder. 

### fzf-lua dependencies
* fzf binary
  I was able to include the `fzf` binary in my Neovim nix package by manipulating links.
  ```nix
  # Runtime dependencies to support Neovim and Neovimthe  plugins
  runtimeDeps = with pkgs; [
    fzf                                     # Fast fuzzy finder: dep of fzf-lua
    ripgrep                                 # Faster more capable grep: dep of fzf-lua
    stylua                                  # Opinionated Lua code formatter
  ];

  symlinkJoin {
    name = appName;                         # Package name being created
    paths = [                               # Paths being symlinked
      neovim-unwrapped                      # Neovim and its files
      pluginPkg                             # Plugin files
    ] ++ runtimeDeps;                       # Include other runtime dependencies
    nativeBuildInputs = [makeWrapper];      # Build tools to make available during build
    ...
    postBuild = ''
      wrapProgram $out/bin/nvim \
        --prefix PATH : ${ lib.makeBinPath runtimeDeps } \
        --add-flags '-u' \
    ... 
  ```
* mini.icons plugin
* lsps plugins

### fzf-lua configuration
```lua
return {
  -- Fast fuzzy finder - better than telescope
  -- depends on mini.icons and optionally lsps
  "fzf-lua",
  cmd = "FzfLua",
  keys = {
    --{ "<leader><leader>", "<cmd>FzfLua buffers<cr>", desc = "Find in open buffers" },
    --{ "<leader>fb", "<cmd>FzfLua builtin<cr>", desc = "Find builtin fuzzy finders" },
    --{ "<leader>ff", "<cmd>FzfLua files<cr>", desc = "Find files in Current Working Directory" },
    --{ "<leader>fg", "<cmd>FzfLua live_grep<cr>", desc = "Find grep through files" },
    --{ "<leader>fh", "<cmd>FzfLua helptags<cr>", desc = "Find help for Neovim" },
    --{ "<leader>fk", "<cmd>FzfLua keymaps<cr>", desc = "Find key maps" },
    --{ "<leader>fr", "<cmd>FzfLua oldfiles<cr>", desc = "Find recent files" },
    { "<leader>fR", "<cmd>FzfLua resume<cr>", desc = "Find resume recent find" },
    { "<leader>fw", "<cmd>FzfLua grep_cword<cr>", desc = "Find word" },

    -- LazyVim bindings, maybe not useful
    { "<leader>ss", "<cmd>FzfLua lsp_document_symbols<cr>", desc = "Find symbol" },
    --{ "<leader>fb", "<cmd>FzfLua buffers<cr>", desc = "Find symbol" },
    { "grr", "<cmd>FzfLua lsp_references<cr>", desc = "Lsp references" },
    { "grt", "<cmd>FzfLua lsp_typedefs<cr>", desc = "Lsp references" },
    { "gd", "<cmd>FzfLua lsp_definitions<cr>", desc = "Lsp definitions" },
    {
      "<leader>sS",
      function()
        require("fzf-lua").lsp_live_workspace_symbols({
          formatter = "path.filename_first",
        })
      end,
      desc = "Find global symbol",
    },
  },
  after = function()
    require("fzf-lua").setup({
      winopts = {
        preview = {
          layout = "vertical",
        },
      },
      fzf_colors = true,
    })
    require("fzf-lua").register_ui_select()
  end,
}
```
