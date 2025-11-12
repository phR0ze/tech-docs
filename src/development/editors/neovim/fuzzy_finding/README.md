# Fuzzy Finding <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

Fuzzy finding provides and easy way to search for files, through buffers, or any other lists and has 
rapidly become the goto for navigating in Neovim. Since I've become interested in upping my Neovim 
game the scene has been dominated first by `telescope` then by `fzf-lua` and now the Snacks plugin 
aggregate provides its own `picker` that provides similar functionality.

### Quick links
- [.. up dir](../README.md)
- [fzf-lua](#fzf-lua)
- [Snacks picker](#snacks-picker)

## Snacks picker
The Snacks picker mimics fzf-lua closely but integrates more tightly with the LazyVim Neovim 
distribution.

### Change picker dimensions

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
