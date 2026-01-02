# LSPs <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

The Language Server Protocol (LSP) was created to allow coding languages to be able to build out 
their own specific functionality that can then be plugged into any editor that supports the LSP 
configuration. Originally developed by Microsoft Visual Studio Code it is now an open standard and 
being rapidly adopted by all IDEs.

### Quick links
- [.. up dir](../README.md)
- [nvim-lspconfig](#nvim-lspconfig)
  - [lspconfig commands](#lspconfig-commands)
- [Rust](#rust)
  - [rustacenvim](#rustacenvim)
  - [rust-analyzer](#rust-analyzer)

## nvim-lspconfig
[nvim-lspconfig](https://github.com/neovim/nvim-lspconfig) makes configuring LSPs a little easier in 
Nvim and is generally the starting place for any LSP configuration in Neovim. It offers a collection 
of LSP server configurations. This can be used in conjunction with [blink.cmp]()

LSP configuration depends on fuzzy finder functionality to surface the lists of options to then 
navigatge between. Telescope, fzf-lua and Snacks picker are all good options. I'll be using 
`Snacks.picker`.

**References**
* [Kickstart.nvim](https://github.com/nvim-lua/kickstart.nvim)

LSP checking usually won't get invoked until after you switch out of insert mode to normal mode so 
its not constantly running while typing.

### Omni Completion
Once you have the LSP up and running you can do `vim.` then press `Ctrl+x`, `Ctrl+o` to invoke omni 
completion. Read about it with `:help ins-completion`

### Get help in installing Lua LSP
1. Launch help with `leader-f-h`
2. Filter on `lspconfig`
3. Switch to list with `Ctrl+l`
4. Select `lspconfig-all`
5. Switch to preview with `Ctrl+p`
6. Search for `lua_ls` and skip to the heading section `lua_ls`

## Rust
Rust can be configured vis `nvim-lspconfig` but also has a plugin dedicated to all things Rust called 
`Rustaceanvim`.

### rustaceanvim
`rustacenvim` is a Neovim plugin that sets up Rust development easier. It handles everything 
separately from nvim-lspconfig at least from a manual configuration perspective.

It interacts with and uses:
* `nvim-lspconfig` - lsp configuration plugin
* `rust-analyzer` - LSP for Rust
* `CodeLLDB` - debugger
* `Vimspector` - plugin
* `nvim-DAP` - Debugger Adaptor Protocol plugin

### rust-analyzer
`rust-analyzer` is the binary that provides Rust LSP integration with various editors

* Code completion
* Some refactoring
* Code analysis and linting
* Go to definition
* Code actions
* Access to documentation
* Show and go to references
* Snippets support
* Bettery syntax highlighting
* Code formatting
