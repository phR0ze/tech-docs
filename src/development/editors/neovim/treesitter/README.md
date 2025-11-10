# Treesitter <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

Treesitter is the goto solution in Neovim for advanced syntax highlighting and code support. It's a 
small C library used to parse source code and is a lot faster and more capable than the default regex 
engine that is used for syntax highlighting by default. Treesitter core is already in Neovim.

### Quick links
- [.. up dir](../README.md)
- [Overview](#overview)
  - [Configuration](#configuration)
- [Code Awareness](#code-awareness)
- [Language Parsers](#language-parsers)
  - [List parsers](#list-parsers)
  - [Install parsers](#install-parsers)
  - [Test parsers](#test-parsers)

## Overview
Treesitter uses a different ***parser*** for every language, which needs to be generated via 
`tree-sitter-cli` from a `grammer.js` file, then compilied to a `.so` library that needs to be placed 
in neovim's `runtimepath` typically under `parser/{language}.so`. The parser is used to generate an 
Abstract Syntax Tree (AST) which then can be used to interpret the syntax of the language and provide 
additional meaning.

Queries can then be used to ask questions of the AST or map colors to language nodes of a certain 
kind. Queries are written in files with extension `.scm`.

### Configuration
- `~/.local/share/nvim/site/parser/c.so`
- `~/.config/nvim/parser/lua.so`
- `~/.config/nvim/queries/lua/highlights.scm`

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

## Code Awareness
Once treesitter is installed and configured along with its parsers Neovim becomes code aware. That 
is, it now understands the different parts of the codeing language your using.

### Inspect
You can park your cursor over the top of a keyword and enter the command `:Inspect` to get 
information from Treesitter about the text object as it pertains to your language and how it relates 
to your color scheme.

### InspectTree
Additionally you can open the tree visualizer from treesitter with `:InspectTree` to see the AST live 
and be able to scroll through the code and see the related components in the AST.

## Language parsers
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

### List parsers
You can run the treesitter health check with `:chechealth nvim-treesitter` which shows the list of 
parsers it recognizes.

**Alternatively just list them directly**
```
:lua for _, p in ipairs(require'nvim-treesitter.info'.installed_parsers()) do print(p) end
```

### Install parsers
After trying a number of different ways to load parsers for treesitter the easiest way appears to 
simply load them along side the plugin itself.

Using `symlinkJoin` we can create a new combined package that includes the nvim-treesitter path as 
well as the parser paths in `nvim-treesitter/parser/` this then gets included in my plugins 
installation path at `./result/pack/plugins/opt/nvim-treesitter` and seems to be the default location 
that treesitter wants to store and find parsers.
```nix
(symlinkJoin {
  name = "nvim-treesitter"; # use the same name to match plugin name expectations
  paths = [ nvim-treesitter (nvim-treesitter.withPlugins (x: [
    x.dart
    x.rust
  ])).dependencies];
})
```

### Test parsers
The easiest way to test if your TS parsing is working is first check that it is installed 
[List Parsers](#list-parsers) then disable the default regex syntax highlighting and check if there 
is still highlighting occurring.

**Disable regex highlighting**
In your current buffer run `:syntax off` to see if the default regex syntax highlighting is being 
used. If so you'll see all color get washed from your current view.


