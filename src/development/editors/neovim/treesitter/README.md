# Treesitter <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

Treesitter is the goto solution in Neovim for advanced syntax highlighting and code support. Its a 
small C library used to parse source code and is a lot faster and more capable than the default regex 
engine that is used for syntax highlighting by default.

### Quick links
- [.. up dir](../README.md)
- [Overview](#overview)
  - [Configuration](#configuration)
  - [Parser install directory](#parser-install-directory)
  - [Check health](#check-health)
- [Commands](#commands)

## Overview
Treesitter uses a different ***parser*** for every language, which needs to be generated via 
`tree-sitter-cli` from a `grammer.js` file, then compilied to a `.so` library that needs to be placed 
in neovim's `runtimepath` typically under `parser/{language}.so`. The parser is used to generate a 
Concrete Syntax Tree (CST) which then can be used to interpret the syntax of the language and provide 
additional meaning.

Queries can then be used to ask questions of the CST or map colors to language nodes of a certain 
kind. Queries are written in files with extension `.scm`.

### Configuration
- `~/.local/share/nvim/site/parser/c.so`
- `~/.config/nvim/parser/lua.so`
- `~/.config/nvim/queries/lua/highlights.scm`

**LazyVim configuration**
```lua
require("lazy").setup({
  {"nvim-treesitter/nvim-treesitter", branch = 'master', lazy = false, build = ":TSUpdate"}
})
```

```lua
require'nvim-treesitter.configs'.setup {

  -- A list of parser names, or "all" (the listed parsers MUST always be installed)
  ensure_installed = { "c", "lua", "vim", "vimdoc", "query", "markdown", "markdown_inline" },

  -- Install parsers synchronously (only applied to `ensure_installed`)
  sync_install = false,

  -- Automatically install missing parsers when entering buffer
  -- Recommendation: set to false if you don't have `tree-sitter` CLI installed locally
  auto_install = true,
  ignore_install = { "javascript" },            -- List of parsers to ignore installing (or "all")

  highlight = {
    enable = true,                              -- Turn on the treesitter syntax hilighting
    additional_vim_regex_highlighting = false,  -- Turn off the built in syntax hilighting
  },
  indent = {
    enable = true
  }
}
```
### Parser install directory
Changing the parser install directory can be done with the `parser_install_dir` field passed into 
setup. Any path in the `runtimepath` will be searched for a `parser` folder.

**Show runtime path**
```
:lua print(vim.inspect(vim.api.nvim_list_runtime_paths()))
:lua for _, p in ipairs(vim.api.nvim_list_runtime_paths()) do print(p) end
```

**List installed parsers**
```
:lua for _, p in ipairs(require'nvim-treesitter.info'.installed_parsers()) do print(p) end
```

### Check health
You can run the treesitter health check with `:chechealth nvim-treesitter` which shows the list of 
parsers it recognizes.

**Disable regex highlighting**
In your current buffer run `:syntax off` to see if the default regex syntax highlighting is being 
used. If so you'll see all color get washed from your current view.

## Modules
Modules provide the top-level features of treesitter.

## Commands

### Change parser install directory
```
  -- It MUST be at the beginning of runtimepath. Otherwise the parsers from Neovim itself
  -- is loaded that may not be compatible with the queries from the 'nvim-treesitter' plugin.
  vim.opt.runtimepath:prepend("/some/path/to/store/parsers")

  require'nvim-treesitter.configs'.setup {
    parser_install_dir = "/some/path/to/store/parsers",
    ...
  }
```

### Language parsers
* `:TSBufEnable {module}` enable module or current buffer
* `:TSBufDisable {module}` disable module or current buffer
* `:TSEnable {module} [{ft}]` enable module on every buffer or only for the specified file type
* `:TSDisable {module} [{ft}]` disable module on every buffer or only for the specified file type
* `:TSModuleInfo [{module}]` list information about modules
* `:TSInstallInfo` get a list of all available languages and their installation status.
* `:TSInstall <language_to_install>` 
* `:TSUpdate <language>` ensures a parser is at the version specified the `lockfile.json`
* `:TSUpdate all`
* `:TSUpdate`
