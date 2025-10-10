# Lua <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

Neovim supports Lua for configuring and controlling Nvim.

### Quick links
- [.. up dir](../README.md)
- [Overview](#overview)
- [Run Lua](#run-lua)
  - [From command line](#from-command-line)
  - [From external file](#from-external-file)
  - [From Vimscript](#from-vimscript)
  - [On boot](#from-vimscript)

## Overview
Lua integration with Vim is  provided through different APIs
* VIM APIs provide support via `vim.cmd` and `vim.fn`
* NVIM API written in C for plugins and GUIs `vim.api`
* Additional Lua endpoints exposed via `vim.*` not already mentioned

**References**
* [Neovim Lua guide](https://neovim.io/doc/user/lua-guide.html)

## Run Lua

**Runtimepath**
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

### Cached require
In contrast to `:source` the `require()` function not only searches through all `lua` directories 
under the `runtimepath` but also caches the module on first use such that further `require()` calls 
will just return the cached file. 

To rerun the file, thus requires that you first remove it from the cache manually
```
package.loaded['myluamodule'] = nil -- remove it from the cache
require('myluamodule')              -- read and execute the module again from disk
```

### From command line
Each `:lua` command has its own scope and variables declared with the local keyword

```
: lua print("Hello!")
```

### From External file
```
:source ~/my/lua/file.lua
```

### From Vimscript
Put it in a here doc
```
lua << EOF
  local tbl = {1, 2, 3}
  for k, v in ipairs(tbl) do
    print(v)
  end
EOF
```
