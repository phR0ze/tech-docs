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

### Information
Find out information about the LSP including what buffers its currently attached to.

```
:LspInfo
```

## Rust

### rustaceanvim
`rustacenvim` is a Neovim plugin that sets up Rust development easier. It interacts with and uses:
* `lspconfig` - lsp configuration plugin
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
