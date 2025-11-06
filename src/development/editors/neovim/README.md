# NeoVim <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />
NeoVim is a fork of Vim aiming to improve user experience and plugin implementation.

### Quick links
- [.. up dir](../README.md)
* [One offs](#one-offs)
  * [Configuration](#configuration)
  * [Config location](#config-location)
  * [Check version](#check-version)
* [Getting started](#getting-started)
  * [Install NeoVim](#install-neovim)
  * [Migrating to NeoVim](#migrating-to-neovim)
* [Config Locations](#config-locations)
* [Plugin Managers](#plugin-managers)
  * [Lazy.nvim](#lazy-nvim)
* [Plugins](#plugins)
  * [Install plugins](#install-plugins)
  * [Plugins](#plugins)
* [Ctrl+tab](#ctrl+tab)
* [Command Syntax](#command-syntax)

### Linked pages
* [Lua](lua/README.md)
* [Lz-n](lz-n/README.md)
* [Runtimepath](runtimepath/README.md)
* [Treesitter](treesitter/README.md)
* [Word Processing](word_processing/README.md)

## Tips

### Configuration
Nvim supports using the `~/.config/nvim/init.lua` as your configuration file. To run any other Lua 
script on startup automatically then you can put it in `~/.config/nvim/plugin/`.

**Runtime path**
```bash
$XDG_CONFIG_HOME/nvim
$XDG_CONFIG_HOME/nvim/after
$XDG_DATA_HOME/nvim/site
$XDG_DATA_HOME/nvim/site/after
$VIMRUNTIME
```

**Example structure path:**
```
~/.config/nvim
|-- after/
|-- ftplugin/
|-- lua/
|   |-- myluamodule.lua
|   |-- other_modules/
|       |-- anothermodule.lua
|       |-- init.lua
|-- plugin/
|-- syntax/
|-- init.vim
```

* Lua code `require("myluamodule")` would load `~/.confi/nvim/lua/myluamodule.lua`
* Lua code `require("other_modules/anothermodule")` would load `~/.confi/nvim/lua/other_modules/anothermodule.lua`
* Lua code `require("other_modules")` would load `~/.confi/nvim/lua/other_modules/init.lua`

### Print runtime paths
```bash
$ nvim --headless +'echo &runtimepath' +q
```

### Loaded script names
```
: scriptnames
```

### Config location
```
:echo stdpath('config')
```

### Check version
Open vim and enter the `:version` command to see version and LuaJIT support
```
NVIM v0.11.3
Build type: Release
LuaJIT 2.1.1741730670
Run ":verbose version" for more info
```

## Getting started

### Configuration
Nvim supports 

### Install NeoVim
```nix
programs.neovim = {
  enable = true;
  defaultEditor = true;
  viAlias = true;
  vimAlias = true;
  configure = ''
    set number
    set cc=80
    set list
    set listchars=tab:→\ ,space:·,nbsp:␣,trail:•,eol:¶,precedes:«,extends:»
    if &diff
      colorscheme blue
    endif
  '';
};
```

### Migrating to NeoVim
Simply copy your `~/.vimrc` to `~/.config/nvim/init.vim` as a starting point

## Config Locations
* ***Config file location:*** `~/.config/nvim/init.vim`
* ***Global user config file location:*** `/etc/xdg/nvim/sysinit.vim`
* ***Global default config file location:*** `/usr/share/nvim/sysinit.vim`
* ***Data directory for swap files:*** `~/.local/share/nvim`

## Plugin Managers
The latest vim plugin manager push is leaning into Lua, performance and fine-grained load control. 
They require newer versions of Neovim and support automatic lazy loading of plugins.

* [lz.n](https://github.com/lumen-oss/lz.n)
  * minimal lazy loading library *not* a plugin manager 
    * events, filetype, key mappings, user commands, colorscheme events
  * does not handle
    * plugin installs
    * dependency management
  * Favored by Nix users as plugin managment is done with Nix
* [lazy.nvim](https://lazy.folke.io/)
  * automatic plugin installation
  * partial git clones
  * lazy load by event, filetype, command, key mappings, etc..
  * dependency management ordering for loading

### Lazy.nvim
[Lazy.nvim](https://www.lazyvim.org/) is both a plugin manager and a distribution of Vim.

### vim-plug
[vim-plug](https://github.com/junegunn/vim-plug) is a minimalist vim plugin manager with super fast
parallel installation/update. It is the most popular one right now as well.

**Install**
```bash
$ curl -fLo ~/.config/nvim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

**Install plugins**
```
$ nvim ~/.config/nvim/init.vim
:PlugInstall
:PlugUpdate
:q
```

## Plugins
Vim plugins in the `vim-plug` world are just github repositories.
* [inspirational list of language plugins](https://github.com/sheerun/vim-polyglot)

### Utilities
```
  # Utilities
  aserebryakov/vim-todo-lists         # Manage TODOs
  #Plug  yegappan/mru
  #Plug  junegunn/fzf , {  dir :  ~/.fzf ,  do :  ./install --all   }
  #Plug  junegunn/fzf.vim
  #Plug  ctrlpvim/ctrlp.vim
  #
  #Plug  Shougo/neocomplete.vim
  #Plug  tommcdo/vim-exchange
  #Plug  ntpeters/vim-better-whitespace
  #Plug  tpope/vim-surround
  #Plug  tpope/vim-repeat
  #Plug  jiangmiao/auto-pairs
  #Plug  vim-scripts/CursorLineCurrentWindow
  #Plug  victormours/better-writing.vim
  #Plug  janko-m/vim-test
  #Plug  skywind3000/asyncrun.vim
  #Plug  w0rp/ale
  #Plugin  scrooloose/nerdtree
  #Plugin  majutsushi/tagbar
  #Plugin  ervandew/supertab
  #Plugin  BufOnly.vim
  #Plugin  wesQ3/vim-windowswap
  #Plugin  SirVer/ultisnips
  #Plugin  junegunn/fzf.vim
  #Plugin  junegunn/fzf
  #Plugin  godlygeek/tabular
  #Plugin  ctrlpvim/ctrlp.vim
  #Plugin  benmills/vimux
  #Plugin  jeetsukumaran/vim-buffergator
  #Plugin  gilsondev/searchtasks.vim
  #Plugin  Shougo/neocomplete.vim
  #Plugin  tpope/vim-dispatch

  # Interface
  scrooloose/nerdtree                 # File explorer sidebar
  vim-airline/vim-airline             # Awesome status bar at bottom with git support
  vim-airline/vim-airline-themes      # Vim Airline themes
  ryanoasis/vim-devicons              # Sweet folder/file icons for nerd tree

  # ColorSchemes
  vim-scripts/CycleColor  			      # Color scheme cycler
  ajmwagar/vim-deus  				          # deus
  YorickPeterse/happy_hacking.vim     # happy_hacking
  w0ng/vim-hybrid  				            # hybrid
  kristijanhusak/vim-hybrid-material  # hybrid_material
  nanotech/jellybeans.vim  			      # jellybeans
  dikiaap/minimalist  				        # minimalist
  marcopaganini/termschool-vim-theme  # termschool

  # Programming
  airblade/vim-gitgutter              # Git integration in gutter
  tpope/vim-fugitive                  # Git integration
  #Plugin  kablamo/vim-git-log
  #Plugin  gregsexton/gitv
  #Plugin  jakedouglas/exuberant-ctags
  #Plugin  honza/vim-snippets
  #Plugin  Townk/vim-autoclose
  #Plugin  tomtom/tcomment_vim
  #Plugin  tobyS/vmustache
  #Plugin  janko-m/vim-test
  #Plugin  maksimr/vim-jsbeautify
  #Plugin  vim-syntastic/syntastic
  #Plugin  neomake/neomake
  #Plugin  artur-shaik/vim-javacomplete2
  #Bundle  jalcine/cmake.vim

  # Syntax highlighting
  zzeroo/vim-vala
  stephpy/vim-yaml                   # yaml
  hail2u/vim-css3-syntax             # css3
  kurayama/systemd-vim-syntax        # systemd

  # Markdown / Writting
  #Plugin  reedes/vim-pencil
  #Plugin  tpope/vim-markdown
  #Plugin  jtratner/vim-flavored-markdown
  #Plugin  LanguageTool

  # HTML
  #Plug  mattn/emmet-vim
  #Plug  slim-template/vim-slim
  #Plug  mustache/vim-mustache-handlebars

  # Javascript
  #Plug  pangloss/vim-javascript
  #Plug  mxw/vim-jsx
  #Plug  othree/yajs.vim
  #Plug  othree/javascript-libraries-syntax.vim
  #Plug  claco/jasmine.vim
  #Plug  kchmck/vim-coffee-script
  #Plug  lfilho/cosco.vim

  # Ruby
  #Plug  Keithbsmiley/rspec.vim
  #Plug  tpope/vim-rails
  #Plug  tpope/vim-endwise
  #Plug  ecomba/vim-ruby-refactoring
  #Plug  vim-ruby/vim-ruby
  #Plug  emilsoman/spec-outline.vim
  #Plug  victormours/vim-rspec
  #Plug  nelstrom/vim-textobj-rubyblock
  #Plug  kana/vim-textobj-user
  #Plug  jgdavey/vim-blockle
  #Plug  KurtPreston/vim-autoformat-rails
  #Plug  ngmy/vim-rubocop

  # Colorize last to ensure overriding
  phR0ze/vim-colorize                # Colorize various plugins
)
```

## Command Syntax

## Ctrl+tab
```
" Buffer controls
" Tab for next buffer, Shift+Tab for previous buffer
" <leader>b will list all buffers with names
" <leader>d will close the current buffer
nnoremap  <silent>   <tab>  :if &modifiable && !&readonly && &modified <cr> :write<cr> :endif<cr>:bnext<cr>
nnoremap  <silent> <s-tab>  :if &modifiable && !&readonly && &modified <cr> :write<cr> :endif<cr>:bprevious<cr>
nnoremap <leader>b :buffers<cr>:b<space> 
nnoremap <leader>d :bdelete<cr>
```

<!-- 
vim: ts=2:sw=2:sts=2
-->
